# Helm Charts Tutorial - Part 3: Real-World Examples and Best Practices

## Overview

This part provides complete real-world examples of Helm charts for production deployments, including Docker integration, Kubernetes best practices, CI/CD integration, and production-ready configurations.

---

## 1. Complete Example: Web Application with Database

### 1.1 Application Overview

**Stack**:
- Frontend: React (Nginx)
- Backend: Node.js API
- Database: PostgreSQL
- Cache: Redis
- Message Queue: RabbitMQ

### 1.2 Chart Structure

```
webapp/
├── Chart.yaml
├── values.yaml
├── values-production.yaml
├── values-staging.yaml
├── .helmignore
├── charts/                    # Dependencies
│   ├── postgresql-12.1.2.tgz
│   ├── redis-17.3.7.tgz
│   └── rabbitmq-11.1.0.tgz
├── templates/
│   ├── _helpers.tpl
│   ├── frontend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── ingress.yaml
│   ├── backend/
│   │   ├── deployment.yaml
│   │   ├── service.yaml
│   │   └── hpa.yaml
│   ├── configmap.yaml
│   ├── secrets.yaml
│   └── hooks/
│       ├── pre-install-db-migration.yaml
│       └── post-install-health-check.yaml
└── files/
    └── nginx.conf
```

### 1.3 Chart.yaml

```yaml
apiVersion: v2
name: webapp
description: Full-stack web application
type: application
version: 1.0.0
appVersion: "2.0.0"
keywords:
  - webapp
  - nodejs
  - react
  - postgresql
maintainers:
  - name: DevOps Team
    email: devops@example.com

dependencies:
  - name: postgresql
    version: "12.1.2"
    repository: "https://charts.bitnami.com/bitnami"
    condition: dependencies.postgresql.enabled
  - name: redis
    version: "17.3.7"
    repository: "https://charts.bitnami.com/bitnami"
    condition: dependencies.redis.enabled
  - name: rabbitmq
    version: "11.1.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: dependencies.rabbitmq.enabled
```

### 1.4 values.yaml

```yaml
# Global settings
global:
  environment: development
  domain: example.com

# Frontend configuration
frontend:
  enabled: true
  replicaCount: 2
  image:
    repository: myregistry/webapp-frontend
    pullPolicy: IfNotPresent
    tag: "2.0.0"
  service:
    type: ClusterIP
    port: 80
  ingress:
    enabled: true
    className: "nginx"
    annotations:
      cert-manager.io/cluster-issuer: "letsencrypt-prod"
      nginx.ingress.kubernetes.io/ssl-redirect: "true"
    hosts:
      - host: app.example.com
        paths:
          - path: /
            pathType: Prefix
    tls:
      - secretName: webapp-frontend-tls
        hosts:
          - app.example.com
  resources:
    limits:
      cpu: 200m
      memory: 256Mi
    requests:
      cpu: 100m
      memory: 128Mi

# Backend configuration
backend:
  enabled: true
  replicaCount: 3
  image:
    repository: myregistry/webapp-backend
    pullPolicy: IfNotPresent
    tag: "2.0.0"
  service:
    type: ClusterIP
    port: 3000
    targetPort: 3000
  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 10
    targetCPUUtilizationPercentage: 70
    targetMemoryUtilizationPercentage: 80
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi
    requests:
      cpu: 500m
      memory: 512Mi
  env:
    - name: NODE_ENV
      value: "production"
    - name: PORT
      value: "3000"
    - name: DATABASE_URL
      valueFrom:
        secretKeyRef:
          name: webapp-db-secret
          key: url
    - name: REDIS_URL
      valueFrom:
        secretKeyRef:
          name: webapp-redis-secret
          key: url
    - name: RABBITMQ_URL
      valueFrom:
        secretKeyRef:
          name: webapp-rabbitmq-secret
          key: url

# Database configuration
database:
  host: postgresql
  port: 5432
  name: webapp
  user: webapp_user
  migrations:
    enabled: true
    image: myregistry/webapp-migrations
    tag: "2.0.0"

# Dependencies
dependencies:
  postgresql:
    enabled: true
  redis:
    enabled: true
  rabbitmq:
    enabled: true

# Override dependency values
postgresql:
  auth:
    postgresPassword: "changeme"
    database: "webapp"
    username: "webapp_user"
    password: "changeme"
  persistence:
    enabled: true
    size: 50Gi
    storageClass: "fast-ssd"
  resources:
    limits:
      cpu: 2000m
      memory: 4Gi
    requests:
      cpu: 1000m
      memory: 2Gi

redis:
  auth:
    enabled: true
    password: "changeme"
  master:
    persistence:
      enabled: true
      size: 20Gi
    resources:
      limits:
        cpu: 1000m
        memory: 2Gi
      requests:
        cpu: 500m
        memory: 1Gi

rabbitmq:
  auth:
    username: "webapp"
    password: "changeme"
  persistence:
    enabled: true
    size: 10Gi
  resources:
    limits:
      cpu: 1000m
      memory: 1Gi
    requests:
      cpu: 500m
      memory: 512Mi
```

### 1.5 Frontend Deployment

```yaml
# templates/frontend/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "webapp.fullname" . }}-frontend
  labels:
    {{- include "webapp.labels" . | nindent 4 }}
    component: frontend
spec:
  replicas: {{ .Values.frontend.replicaCount }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
  selector:
    matchLabels:
      {{- include "webapp.selectorLabels" . | nindent 6 }}
      component: frontend
  template:
    metadata:
      labels:
        {{- include "webapp.selectorLabels" . | nindent 8 }}
        component: frontend
      annotations:
        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
    spec:
      {{- if .Values.image.pullSecrets }}
      imagePullSecrets:
        {{- range .Values.image.pullSecrets }}
        - name: {{ . }}
        {{- end }}
      {{- end }}
      containers:
      - name: frontend
        image: "{{ .Values.frontend.image.repository }}:{{ .Values.frontend.image.tag }}"
        imagePullPolicy: {{ .Values.frontend.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        resources:
          {{- toYaml .Values.frontend.resources | nindent 10 }}
        volumeMounts:
        - name: nginx-config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
      volumes:
      - name: nginx-config
        configMap:
          name: {{ include "webapp.fullname" . }}-nginx-config
```

### 1.6 Backend Deployment with HPA

```yaml
# templates/backend/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "webapp.fullname" . }}-backend
  labels:
    {{- include "webapp.labels" . | nindent 4 }}
    component: backend
spec:
  replicas: {{ .Values.backend.replicaCount }}
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      {{- include "webapp.selectorLabels" . | nindent 6 }}
      component: backend
  template:
    metadata:
      labels:
        {{- include "webapp.selectorLabels" . | nindent 8 }}
        component: backend
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "3000"
        prometheus.io/path: "/metrics"
    spec:
      serviceAccountName: {{ include "webapp.serviceAccountName" . }}
      {{- if .Values.image.pullSecrets }}
      imagePullSecrets:
        {{- range .Values.image.pullSecrets }}
        - name: {{ . }}
        {{- end }}
      {{- end }}
      containers:
      - name: backend
        image: "{{ .Values.backend.image.repository }}:{{ .Values.backend.image.tag }}"
        imagePullPolicy: {{ .Values.backend.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.backend.service.targetPort }}
          protocol: TCP
        env:
        {{- range .Values.backend.env }}
        - name: {{ .name }}
          {{- if .value }}
          value: {{ .value | quote }}
          {{- end }}
          {{- if .valueFrom }}
          valueFrom:
            {{- toYaml .valueFrom | nindent 12 }}
          {{- end }}
        {{- end }}
        livenessProbe:
          httpGet:
            path: /health
            port: http
          initialDelaySeconds: 60
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: http
          initialDelaySeconds: 30
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
        resources:
          {{- toYaml .Values.backend.resources | nindent 10 }}
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        fsGroup: 1000
```

```yaml
# templates/backend/hpa.yaml
{{- if .Values.backend.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "webapp.fullname" . }}-backend
  labels:
    {{- include "webapp.labels" . | nindent 4 }}
    component: backend
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "webapp.fullname" . }}-backend
  minReplicas: {{ .Values.backend.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.backend.autoscaling.maxReplicas }}
  metrics:
  {{- if .Values.backend.autoscaling.targetCPUUtilizationPercentage }}
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: {{ .Values.backend.autoscaling.targetCPUUtilizationPercentage }}
  {{- end }}
  {{- if .Values.backend.autoscaling.targetMemoryUtilizationPercentage }}
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: {{ .Values.backend.autoscaling.targetMemoryUtilizationPercentage }}
  {{- end }}
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 60
      - type: Pods
        value: 2
        periodSeconds: 60
      selectPolicy: Max
{{- end }}
```

### 1.7 Database Migration Hook

```yaml
# templates/hooks/pre-install-db-migration.yaml
{{- if .Values.database.migrations.enabled }}
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "webapp.fullname" . }}-db-migration
  labels:
    {{- include "webapp.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": pre-install,pre-upgrade
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
spec:
  template:
    metadata:
      labels:
        {{- include "webapp.selectorLabels" . | nindent 8 }}
    spec:
      restartPolicy: Never
      containers:
      - name: migration
        image: "{{ .Values.database.migrations.image }}:{{ .Values.database.migrations.tag }}"
        imagePullPolicy: IfNotPresent
        command:
        - /bin/sh
        - -c
        - |
          echo "Running database migrations..."
          npm run migrate:up
          echo "Migrations completed successfully"
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: {{ include "webapp.fullname" . }}-db-secret
              key: url
        - name: NODE_ENV
          value: "production"
        resources:
          limits:
            cpu: 500m
            memory: 512Mi
          requests:
            cpu: 100m
            memory: 128Mi
{{- end }}
```

---

## 2. Docker Integration

### 2.1 Multi-Stage Dockerfile

```dockerfile
# Dockerfile for Backend
# Stage 1: Build
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build application
RUN npm run build

# Stage 2: Runtime
FROM node:18-alpine

WORKDIR /app

# Create non-root user
RUN addgroup -g 1000 nodeuser && \
    adduser -D -u 1000 -G nodeuser nodeuser

# Copy built application
COPY --from=builder --chown=nodeuser:nodeuser /app/dist ./dist
COPY --from=builder --chown=nodeuser:nodeuser /app/node_modules ./node_modules
COPY --from=builder --chown=nodeuser:nodeuser /app/package*.json ./

# Switch to non-root user
USER nodeuser

# Expose port
EXPOSE 3000

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node healthcheck.js

# Start application
CMD ["node", "dist/index.js"]
```

```dockerfile
# Dockerfile for Frontend
# Stage 1: Build
FROM node:18-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source code
COPY . .

# Build application
RUN npm run build

# Stage 2: Nginx
FROM nginx:1.21-alpine

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Copy built files
COPY --from=builder /app/build /usr/share/nginx/html

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/health || exit 1

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

### 2.2 Docker Compose for Local Development

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile.dev
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
    environment:
      - REACT_APP_API_URL=http://backend:3000

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile.dev
    ports:
      - "3001:3000"
    volumes:
      - ./backend:/app
      - /app/node_modules
    environment:
      - NODE_ENV=development
      - DATABASE_URL=postgresql://webapp:password@postgres:5432/webapp
      - REDIS_URL=redis://redis:6379
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:14-alpine
    environment:
      - POSTGRES_USER=webapp
      - POSTGRES_PASSWORD=password
      - POSTGRES_DB=webapp
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### 2.3 Building and Pushing Images

```bash
#!/bin/bash
# build-and-push.sh

set -e

REGISTRY="myregistry.com"
VERSION="${1:-latest}"

# Build and push backend
echo "Building backend..."
docker build -t ${REGISTRY}/webapp-backend:${VERSION} ./backend
docker push ${REGISTRY}/webapp-backend:${VERSION}

# Build and push frontend
echo "Building frontend..."
docker build -t ${REGISTRY}/webapp-frontend:${VERSION} ./frontend
docker push ${REGISTRY}/webapp-frontend:${VERSION}

# Build and push migrations
echo "Building migrations..."
docker build -t ${REGISTRY}/webapp-migrations:${VERSION} ./migrations
docker push ${REGISTRY}/webapp-migrations:${VERSION}

echo "All images built and pushed successfully!"
```

### 2.4 Using Images in Helm

```yaml
# values.yaml
frontend:
  image:
    repository: myregistry.com/webapp-frontend
    tag: "2.0.0"
    pullPolicy: Always
    pullSecrets:
      - registry-secret

backend:
  image:
    repository: myregistry.com/webapp-backend
    tag: "2.0.0"
    pullPolicy: Always
    pullSecrets:
      - registry-secret

database:
  migrations:
    image: myregistry.com/webapp-migrations
    tag: "2.0.0"
```

---

## 3. CI/CD Integration

### 3.1 GitHub Actions Workflow

```yaml
# .github/workflows/deploy.yml
name: Build and Deploy

on:
  push:
    branches:
      - main
      - develop
  pull_request:
    branches:
      - main

env:
  REGISTRY: myregistry.com
  HELM_VERSION: "3.12.0"

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Log in to Registry
        uses: docker/login-action@v2
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ secrets.REGISTRY_USERNAME }}
          password: ${{ secrets.REGISTRY_PASSWORD }}

      - name: Build and push backend
        uses: docker/build-push-action@v4
        with:
          context: ./backend
          push: true
          tags: |
            ${{ env.REGISTRY }}/webapp-backend:${{ github.sha }}
            ${{ env.REGISTRY }}/webapp-backend:latest
          cache-from: type=registry,ref=${{ env.REGISTRY }}/webapp-backend:buildcache
          cache-to: type=registry,ref=${{ env.REGISTRY }}/webapp-backend:buildcache,mode=max

      - name: Build and push frontend
        uses: docker/build-push-action@v4
        with:
          context: ./frontend
          push: true
          tags: |
            ${{ env.REGISTRY }}/webapp-frontend:${{ github.sha }}
            ${{ env.REGISTRY }}/webapp-frontend:latest

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    steps:
      - uses: actions/checkout@v3

      - name: Install Helm
        uses: azure/setup-helm@v3
        with:
          version: ${{ env.HELM_VERSION }}

      - name: Configure kubectl
        uses: azure/setup-kubectl@v3

      - name: Set up kubeconfig
        run: |
          echo "${{ secrets.KUBECONFIG_STAGING }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig

      - name: Deploy to Staging
        run: |
          helm upgrade --install webapp ./helm/webapp \
            --namespace staging \
            --create-namespace \
            --set frontend.image.tag=${{ github.sha }} \
            --set backend.image.tag=${{ github.sha }} \
            --set database.migrations.tag=${{ github.sha }} \
            --values ./helm/webapp/values-staging.yaml \
            --wait \
            --timeout 5m

  deploy-production:
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    steps:
      - uses: actions/checkout@v3

      - name: Install Helm
        uses: azure/setup-helm@v3
        with:
          version: ${{ env.HELM_VERSION }}

      - name: Configure kubectl
        uses: azure/setup-kubectl@v3

      - name: Set up kubeconfig
        run: |
          echo "${{ secrets.KUBECONFIG_PRODUCTION }}" | base64 -d > kubeconfig
          export KUBECONFIG=kubeconfig

      - name: Deploy to Production
        run: |
          helm upgrade --install webapp ./helm/webapp \
            --namespace production \
            --create-namespace \
            --set frontend.image.tag=${{ github.sha }} \
            --set backend.image.tag=${{ github.sha }} \
            --set database.migrations.tag=${{ github.sha }} \
            --values ./helm/webapp/values-production.yaml \
            --wait \
            --timeout 10m

      - name: Run smoke tests
        run: |
          kubectl run smoke-test --rm -i --restart=Never \
            --image=curlimages/curl:latest \
            -- curl -f http://webapp-frontend/health
```

### 3.2 GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - build
  - test
  - deploy

variables:
  REGISTRY: myregistry.com
  HELM_VERSION: "3.12.0"

build-backend:
  stage: build
  script:
    - docker build -t ${REGISTRY}/webapp-backend:${CI_COMMIT_SHA} ./backend
    - docker push ${REGISTRY}/webapp-backend:${CI_COMMIT_SHA}
  only:
    - main
    - develop

build-frontend:
  stage: build
  script:
    - docker build -t ${REGISTRY}/webapp-frontend:${CI_COMMIT_SHA} ./frontend
    - docker push ${REGISTRY}/webapp-frontend:${CI_COMMIT_SHA}
  only:
    - main
    - develop

deploy-staging:
  stage: deploy
  image: alpine/helm:${HELM_VERSION}
  script:
    - helm upgrade --install webapp ./helm/webapp \
        --namespace staging \
        --create-namespace \
        --set frontend.image.tag=${CI_COMMIT_SHA} \
        --set backend.image.tag=${CI_COMMIT_SHA} \
        --values ./helm/webapp/values-staging.yaml \
        --wait
  environment:
    name: staging
    url: https://staging.example.com
  only:
    - develop

deploy-production:
  stage: deploy
  image: alpine/helm:${HELM_VERSION}
  script:
    - helm upgrade --install webapp ./helm/webapp \
        --namespace production \
        --create-namespace \
        --set frontend.image.tag=${CI_COMMIT_SHA} \
        --set backend.image.tag=${CI_COMMIT_SHA} \
        --values ./helm/webapp/values-production.yaml \
        --wait
  environment:
    name: production
    url: https://app.example.com
  when: manual
  only:
    - main
```

---

## 4. Production Best Practices

### 4.1 Security

#### Security Context

```yaml
# templates/backend/deployment.yaml
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 1000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: backend
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
          - ALL
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: var-run
      mountPath: /var/run
  volumes:
  - name: tmp
    emptyDir: {}
  - name: var-run
    emptyDir: {}
```

#### Network Policies

```yaml
# templates/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: {{ include "webapp.fullname" . }}-backend
spec:
  podSelector:
    matchLabels:
      component: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          component: frontend
    ports:
    - protocol: TCP
      port: 3000
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: postgresql
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
```

### 4.2 Resource Management

```yaml
# values-production.yaml
backend:
  resources:
    limits:
      cpu: 2000m
      memory: 2Gi
    requests:
      cpu: 1000m
      memory: 1Gi
  autoscaling:
    enabled: true
    minReplicas: 5
    maxReplicas: 20
    targetCPUUtilizationPercentage: 70
    targetMemoryUtilizationPercentage: 80

frontend:
  resources:
    limits:
      cpu: 500m
      memory: 512Mi
    requests:
      cpu: 250m
      memory: 256Mi
```

### 4.3 Monitoring and Observability

```yaml
# templates/service-monitor.yaml (Prometheus)
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: {{ include "webapp.fullname" . }}-backend
  labels:
    {{- include "webapp.labels" . | nindent 4 }}
spec:
  selector:
    matchLabels:
      component: backend
  endpoints:
  - port: http
    path: /metrics
    interval: 30s
```

### 4.4 Backup and Disaster Recovery

```yaml
# templates/backup-job.yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: {{ include "webapp.fullname" . }}-backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:14-alpine
            command:
            - /bin/sh
            - -c
            - |
              PGPASSWORD=$DB_PASSWORD pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME | \
              gzip > /backup/backup-$(date +%Y%m%d-%H%M%S).sql.gz
              aws s3 cp /backup/*.sql.gz s3://${BACKUP_BUCKET}/backups/
              # Keep only last 30 days
              find /backup -name "*.sql.gz" -mtime +30 -delete
            env:
            - name: DB_HOST
              value: {{ .Values.database.host }}
            - name: DB_USER
              value: {{ .Values.database.user }}
            - name: DB_NAME
              value: {{ .Values.database.name }}
            - name: DB_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ include "webapp.fullname" . }}-db-secret
                  key: password
            - name: BACKUP_BUCKET
              value: {{ .Values.backup.bucket }}
            volumeMounts:
            - name: backup
              mountPath: /backup
          volumes:
          - name: backup
            emptyDir: {}
          restartPolicy: OnFailure
```

---

## 5. Deployment Strategies

### 5.1 Blue-Green Deployment

```yaml
# values-blue-green.yaml
deployment:
  strategy: blue-green
  active: blue
  blue:
    image:
      tag: "2.0.0"
  green:
    image:
      tag: "2.1.0"
```

### 5.2 Canary Deployment

```yaml
# values-canary.yaml
deployment:
  strategy: canary
  canary:
    enabled: true
    weight: 10  # 10% traffic
    image:
      tag: "2.1.0"
  stable:
    image:
      tag: "2.0.0"
```

---

## 6. Troubleshooting

### 6.1 Common Issues

**Issue 1: Chart installation fails**

```bash
# Debug installation
helm install webapp ./webapp --dry-run --debug

# Check logs
kubectl logs -l app=webapp

# Describe resources
kubectl describe pod <pod-name>
```

**Issue 2: Image pull errors**

```bash
# Check image pull secrets
kubectl get secrets

# Test image pull
kubectl run test --image=myregistry.com/webapp-backend:2.0.0 --rm -it --restart=Never
```

**Issue 3: Database connection issues**

```bash
# Check database service
kubectl get svc postgresql

# Test connection
kubectl run test --image=postgres:14-alpine --rm -it --restart=Never -- \
  psql -h postgresql -U webapp_user -d webapp
```

### 6.2 Debugging Commands

```bash
# Get release status
helm status webapp

# View release history
helm history webapp

# Rollback
helm rollback webapp 1

# Get rendered templates
helm template webapp ./webapp

# Validate values
helm lint ./webapp
```

---

## 7. Complete Deployment Example

### 7.1 Deployment Script

```bash
#!/bin/bash
# deploy.sh

set -e

ENVIRONMENT=${1:-staging}
VERSION=${2:-latest}

echo "Deploying to ${ENVIRONMENT} with version ${VERSION}"

# Update dependencies
helm dependency update ./webapp

# Build dependencies
helm dependency build ./webapp

# Lint chart
helm lint ./webapp

# Deploy
helm upgrade --install webapp ./webapp \
  --namespace ${ENVIRONMENT} \
  --create-namespace \
  --values ./webapp/values-${ENVIRONMENT}.yaml \
  --set frontend.image.tag=${VERSION} \
  --set backend.image.tag=${VERSION} \
  --wait \
  --timeout 10m

# Run tests
helm test webapp --namespace ${ENVIRONMENT}

echo "Deployment completed successfully!"
```

### 7.2 Usage

```bash
# Deploy to staging
./deploy.sh staging 2.0.0

# Deploy to production
./deploy.sh production 2.0.0
```

---

## Summary

**Key Takeaways**:
1. ✅ Helm simplifies complex Kubernetes deployments
2. ✅ Docker integration is seamless
3. ✅ CI/CD pipelines automate deployments
4. ✅ Production best practices ensure reliability
5. ✅ Monitoring and observability are critical
6. ✅ Security should be built-in from the start

**Best Practices**:
- Use multi-stage Docker builds
- Implement proper health checks
- Set resource limits and requests
- Use HPA for autoscaling
- Implement network policies
- Set up monitoring and alerting
- Use secrets management
- Implement backup strategies
- Test charts before deployment
- Document everything

This completes the comprehensive Helm Charts tutorial covering fundamentals, advanced features, and real-world production deployments!

