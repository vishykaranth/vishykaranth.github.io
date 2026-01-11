# Helm Charts Tutorial - Part 2: Advanced Features

## Overview

This part covers advanced Helm features including hooks, dependencies, subcharts, library charts, and advanced templating techniques for complex Kubernetes deployments.

---

## 1. Helm Hooks

### 1.1 What are Hooks?

**Hooks** are Kubernetes resources that run at specific points in a release's lifecycle. They allow you to perform actions before or after:
- Installing a chart
- Upgrading a chart
- Deleting a chart
- Rolling back a chart

### 1.2 Hook Lifecycle

```
┌─────────────────────────────────────────┐
│         HELM HOOK LIFECYCLE             │
└─────────────────────────────────────────┘

Install:
1. pre-install
2. Install resources
3. post-install

Upgrade:
1. pre-upgrade
2. Upgrade resources
3. post-upgrade

Rollback:
1. pre-rollback
2. Rollback resources
3. post-rollback

Delete:
1. pre-delete
2. Delete resources
3. post-delete
```

### 1.3 Hook Annotations

```yaml
metadata:
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": before-hook-creation,hook-succeeded
```

**Hook Types**:
- `pre-install`: Before installation
- `post-install`: After installation
- `pre-upgrade`: Before upgrade
- `post-upgrade`: After upgrade
- `pre-rollback`: Before rollback
- `post-rollback`: After rollback
- `pre-delete`: Before deletion
- `post-delete`: After deletion

**Hook Weight**: Determines execution order (lower = earlier)

**Delete Policy**:
- `before-hook-creation`: Delete before new hook runs
- `hook-succeeded`: Delete if hook succeeds
- `hook-failed`: Delete if hook fails

### 1.4 Example: Database Migration Hook

#### Pre-Install Hook: Database Setup

```yaml
# templates/hooks/pre-install-db-setup.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "my-app.fullname" . }}-db-setup
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": pre-install
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      restartPolicy: Never
      containers:
      - name: db-setup
        image: postgres:14
        command:
        - /bin/sh
        - -c
        - |
          PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -U $DB_USER -d $DB_NAME <<EOF
          CREATE TABLE IF NOT EXISTS users (
            id SERIAL PRIMARY KEY,
            name VARCHAR(100),
            email VARCHAR(100)
          );
          EOF
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
              name: {{ include "my-app.fullname" . }}-db-secret
              key: password
```

#### Post-Install Hook: Health Check

```yaml
# templates/hooks/post-install-health-check.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "my-app.fullname" . }}-health-check
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": post-install
    "helm.sh/hook-weight": "5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      restartPolicy: Never
      containers:
      - name: health-check
        image: curlimages/curl:latest
        command:
        - /bin/sh
        - -c
        - |
          for i in $(seq 1 30); do
            if curl -f http://{{ include "my-app.fullname" . }}:{{ .Values.service.port }}/health; then
              echo "Health check passed"
              exit 0
            fi
            echo "Waiting for service... ($i/30)"
            sleep 10
          done
          echo "Health check failed"
          exit 1
```

### 1.5 Example: Backup Hook

#### Pre-Delete Hook: Backup Data

```yaml
# templates/hooks/pre-delete-backup.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: {{ include "my-app.fullname" . }}-backup
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": pre-delete
    "helm.sh/hook-weight": "-5"
    "helm.sh/hook-delete-policy": hook-succeeded
spec:
  template:
    metadata:
      labels:
        {{- include "my-app.selectorLabels" . | nindent 8 }}
    spec:
      restartPolicy: Never
      containers:
      - name: backup
        image: postgres:14
        command:
        - /bin/sh
        - -c
        - |
          PGPASSWORD=$DB_PASSWORD pg_dump -h $DB_HOST -U $DB_USER -d $DB_NAME > /backup/backup-$(date +%Y%m%d-%H%M%S).sql
          aws s3 cp /backup/*.sql s3://{{ .Values.backup.bucket }}/backups/
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
              name: {{ include "my-app.fullname" . }}-db-secret
              key: password
        - name: AWS_ACCESS_KEY_ID
          valueFrom:
            secretKeyRef:
              name: {{ include "my-app.fullname" . }}-aws-secret
              key: access-key-id
        - name: AWS_SECRET_ACCESS_KEY
          valueFrom:
            secretKeyRef:
              name: {{ include "my-app.fullname" . }}-aws-secret
              key: secret-access-key
        volumeMounts:
        - name: backup
          mountPath: /backup
      volumes:
      - name: backup
        emptyDir: {}
```

---

## 2. Chart Dependencies

### 2.1 What are Dependencies?

**Dependencies** allow you to include other charts as subcharts. This enables:
- Composing complex applications
- Reusing common components
- Managing versioned dependencies

### 2.2 Chart.yaml with Dependencies

```yaml
# Chart.yaml
apiVersion: v2
name: my-app
description: Main application chart
type: application
version: 1.0.0
appVersion: "1.0.0"

dependencies:
  - name: postgresql
    version: "12.1.2"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
  - name: redis
    version: "17.3.7"
    repository: "https://charts.bitnami.com/bitnami"
    condition: redis.enabled
  - name: nginx
    version: "13.2.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: nginx.enabled
```

### 2.3 Managing Dependencies

```bash
# Update dependencies
helm dependency update

# Build dependencies
helm dependency build

# List dependencies
helm dependency list

# This creates:
# - charts/ directory with dependency charts
# - Chart.lock file with locked versions
```

### 2.4 Overriding Dependency Values

```yaml
# values.yaml
postgresql:
  enabled: true
  auth:
    postgresPassword: "mypassword"
    database: "myapp"
  persistence:
    enabled: true
    size: 20Gi

redis:
  enabled: true
  auth:
    enabled: true
    password: "redispassword"
  master:
    persistence:
      enabled: true
      size: 10Gi

nginx:
  enabled: true
  service:
    type: LoadBalancer
```

### 2.5 Example: Multi-Tier Application

#### Chart Structure

```
my-app/
├── Chart.yaml
├── values.yaml
├── charts/              # Dependencies
│   ├── postgresql/
│   ├── redis/
│   └── nginx/
└── templates/
    ├── deployment.yaml
    └── service.yaml
```

#### Chart.yaml

```yaml
apiVersion: v2
name: my-app
description: Multi-tier application
type: application
version: 1.0.0

dependencies:
  - name: postgresql
    version: "12.1.2"
    repository: "https://charts.bitnami.com/bitnami"
    condition: dependencies.postgresql.enabled
  - name: redis
    version: "17.3.7"
    repository: "https://charts.bitnami.com/bitnami"
    condition: dependencies.redis.enabled
```

#### values.yaml

```yaml
# Application values
app:
  image:
    repository: myapp
    tag: "1.0.0"
  replicaCount: 3

# Dependency values
dependencies:
  postgresql:
    enabled: true
  redis:
    enabled: true

# Override dependency values
postgresql:
  auth:
    postgresPassword: "changeme"
    database: "myapp"
  persistence:
    enabled: true
    size: 20Gi

redis:
  auth:
    enabled: true
    password: "changeme"
  master:
    persistence:
      enabled: true
      size: 10Gi
```

---

## 3. Subcharts

### 3.1 What are Subcharts?

**Subcharts** are charts included within a parent chart. They're different from dependencies:
- Dependencies: External charts from repositories
- Subcharts: Charts included in the same repository

### 3.2 Subchart Structure

```
parent-chart/
├── Chart.yaml
├── values.yaml
├── charts/
│   └── child-chart/      # Subchart
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
└── templates/
```

### 3.3 Subchart Values

Parent chart can override subchart values:

```yaml
# parent-chart/values.yaml
child-chart:
  replicaCount: 5
  image:
    tag: "2.0.0"
```

Subchart accesses values:

```yaml
# child-chart/templates/deployment.yaml
spec:
  replicas: {{ .Values.replicaCount }}
  # Parent values accessible via .Values.global
```

### 3.4 Global Values

Values accessible to all charts:

```yaml
# parent-chart/values.yaml
global:
  environment: production
  domain: example.com

child-chart:
  replicaCount: 3
```

All charts can access:

```yaml
{{ .Values.global.environment }}
{{ .Values.global.domain }}
```

---

## 4. Library Charts

### 4.1 What are Library Charts?

**Library charts** provide reusable template definitions that other charts can include. They don't create resources themselves.

### 4.2 Creating a Library Chart

#### Chart.yaml

```yaml
apiVersion: v2
name: common-lib
description: Common library templates
type: library
version: 1.0.0
```

#### templates/_helpers.tpl

```yaml
{{/*
Common labels for all resources
*/}}
{{- define "common-lib.labels" -}}
app.kubernetes.io/managed-by: {{ .Release.Service }}
app.kubernetes.io/part-of: {{ .Chart.Name }}
{{- end }}

{{/*
Resource limits based on tier
*/}}
{{- define "common-lib.resources" -}}
{{- $tier := .Values.tier | default "standard" }}
{{- if eq $tier "small" }}
resources:
  limits:
    cpu: 100m
    memory: 128Mi
  requests:
    cpu: 50m
    memory: 64Mi
{{- else if eq $tier "standard" }}
resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi
{{- else if eq $tier "large" }}
resources:
  limits:
    cpu: 2000m
    memory: 2Gi
  requests:
    cpu: 1000m
    memory: 1Gi
{{- end }}
{{- end }}
```

### 4.3 Using Library Charts

```yaml
# Chart.yaml
dependencies:
  - name: common-lib
    version: "1.0.0"
    repository: "file://../common-lib"
    condition: common-lib.enabled
```

```yaml
# templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    {{- include "common-lib.labels" . | nindent 4 }}
spec:
  template:
    spec:
      containers:
      - name: app
        {{- include "common-lib.resources" . | nindent 8 }}
```

---

## 5. Advanced Templating

### 5.1 Template Functions

#### String Functions

```yaml
# Upper case
{{ .Values.name | upper }}

# Lower case
{{ .Values.name | lower }}

# Title case
{{ .Values.name | title }}

# Truncate
{{ .Values.name | trunc 10 }}

# Replace
{{ .Values.name | replace "old" "new" }}

# Contains
{{- if contains "test" .Values.name }}
# ...
{{- end }}

# HasPrefix/HasSuffix
{{- if hasPrefix "app-" .Values.name }}
# ...
{{- end }}
```

#### Math Functions

```yaml
# Add
{{ add .Values.replicaCount 1 }}

# Subtract
{{ sub .Values.replicaCount 1 }}

# Multiply
{{ mul .Values.replicaCount 2 }}

# Divide
{{ div .Values.total 2 }}

# Modulo
{{ mod .Values.count 3 }}
```

#### List Functions

```yaml
# First
{{ first .Values.items }}

# Last
{{ last .Values.items }}

# Has
{{- if has "item" .Values.items }}
# ...
{{- end }}

# Index
{{ index .Values.items 0 }}

# Uniq
{{ .Values.items | uniq }}

# Sort
{{ .Values.items | sortAlpha }}
```

### 5.2 Control Structures

#### If-Else

```yaml
{{- if .Values.enabled }}
enabled: true
{{- else }}
enabled: false
{{- end }}

# With else-if
{{- if eq .Values.env "production" }}
replicas: 5
{{- else if eq .Values.env "staging" }}
replicas: 2
{{- else }}
replicas: 1
{{- end }}
```

#### With

```yaml
{{- with .Values.database }}
host: {{ .host }}
port: {{ .port }}
{{- end }}

# Equivalent to:
host: {{ .Values.database.host }}
port: {{ .Values.database.port }}
```

#### Range

```yaml
# Range over map
{{- range $key, $value := .Values.env }}
{{ $key }}: {{ $value }}
{{- end }}

# Range over list
{{- range .Values.items }}
- {{ . }}
{{- end }}

# Range with index
{{- range $index, $item := .Values.items }}
{{ $index }}: {{ $item }}
{{- end }}
```

### 5.3 Advanced Patterns

#### Pattern 1: Conditional Resource Creation

```yaml
{{- if .Values.feature.enabled }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "my-app.fullname" . }}-feature-config
data:
  {{- range $key, $value := .Values.feature.config }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
{{- end }}
```

#### Pattern 2: Dynamic Resource Names

```yaml
{{- range $index, $service := .Values.services }}
---
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-app.fullname" $ }}-{{ $service.name }}
spec:
  selector:
    app: {{ $service.name }}
  ports:
  {{- range $service.ports }}
  - port: {{ .port }}
    targetPort: {{ .targetPort }}
  {{- end }}
{{- end }}
```

#### Pattern 3: Merging Values

```yaml
{{- $defaultValues := dict "replicas" 3 "image" "nginx" }}
{{- $userValues := .Values }}
{{- $merged := merge $defaultValues $userValues }}

replicas: {{ $merged.replicas }}
image: {{ $merged.image }}
```

#### Pattern 4: Required Values

```yaml
{{- if not .Values.database.host }}
{{- fail "database.host is required" }}
{{- end }}

# Or using required function
database:
  host: {{ required "database.host is required" .Values.database.host }}
```

---

## 6. Advanced Deployment Patterns

### 6.1 Blue-Green Deployment

```yaml
# values.yaml
deployment:
  strategy: blue-green
  blue:
    image: nginx:1.20
  green:
    image: nginx:1.21
  active: blue
```

```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-app.fullname" . }}
spec:
  selector:
    version: {{ .Values.deployment.active }}
  ports:
  - port: 80
```

```yaml
# templates/deployment-blue.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}-blue
spec:
  replicas: {{ if eq .Values.deployment.active "blue" }}{{ .Values.replicaCount }}{{ else }}0{{ end }}
  template:
    metadata:
      labels:
        version: blue
    spec:
      containers:
      - image: {{ .Values.deployment.blue.image }}
```

### 6.2 Canary Deployment

```yaml
# values.yaml
deployment:
  strategy: canary
  canary:
    enabled: true
    weight: 10  # 10% traffic
  stable:
    image: nginx:1.20
  canary:
    image: nginx:1.21
```

```yaml
# templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-app.fullname" . }}
spec:
  selector:
    app: {{ include "my-app.name" . }}
  ports:
  - port: 80
```

```yaml
# templates/virtualservice.yaml (Istio)
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: {{ include "my-app.fullname" . }}
spec:
  hosts:
  - {{ .Values.ingress.host }}
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: {{ include "my-app.fullname" . }}-canary
      weight: 100
  - route:
    - destination:
        host: {{ include "my-app.fullname" . }}-stable
      weight: {{ sub 100 .Values.deployment.canary.weight }}
    - destination:
        host: {{ include "my-app.fullname" . }}-canary
      weight: {{ .Values.deployment.canary.weight }}
```

---

## 7. Secrets Management

### 7.1 Creating Secrets

```yaml
# templates/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "my-app.fullname" . }}-secret
type: Opaque
data:
  username: {{ .Values.secrets.username | b64enc }}
  password: {{ .Values.secrets.password | b64enc }}
stringData:
  config.yaml: |
    database:
      host: {{ .Values.database.host }}
      port: {{ .Values.database.port }}
```

### 7.2 External Secrets (Operator)

```yaml
# templates/externalsecret.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: {{ include "my-app.fullname" . }}-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-manager
    kind: SecretStore
  target:
    name: {{ include "my-app.fullname" . }}-secret
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: myapp/database/password
  - secretKey: api-key
    remoteRef:
      key: myapp/api/key
```

---

## 8. ConfigMap Management

### 7.1 Dynamic ConfigMaps

```yaml
# templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "my-app.fullname" . }}-config
data:
  app.properties: |
    {{- range $key, $value := .Values.config }}
    {{ $key }}={{ $value }}
    {{- end }}
  nginx.conf: |
    {{- .Files.Get "files/nginx.conf" | indent 4 }}
```

### 7.2 Multi-ConfigMap Pattern

```yaml
# templates/configmaps.yaml
{{- range $name, $config := .Values.configMaps }}
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "my-app.fullname" $ }}-{{ $name }}-config
data:
  {{- range $key, $value := $config }}
  {{ $key }}: {{ $value | quote }}
  {{- end }}
{{- end }}
```

---

## 9. Testing Advanced Charts

### 9.1 Unit Testing with helm-unittest

Install plugin:

```bash
helm plugin install https://github.com/quintush/helm-unittest
```

Create test file:

```yaml
# tests/deployment_test.yaml
suite: test deployment
templates:
  - deployment.yaml
tests:
  - it: should have correct replica count
    set:
      replicaCount: 3
    asserts:
      - equal:
          path: spec.replicas
          value: 3
  - it: should use correct image
    set:
      image:
        repository: nginx
        tag: "1.21"
    asserts:
      - equal:
          path: spec.template.spec.containers[0].image
          value: nginx:1.21
```

Run tests:

```bash
helm unittest .
```

### 9.2 Integration Testing

```yaml
# templates/tests/integration-test.yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ include "my-app.fullname" . }}-integration-test
  annotations:
    "helm.sh/hook": test
    "helm.sh/hook-weight": "10"
spec:
  containers:
  - name: test
    image: curlimages/curl:latest
    command:
    - /bin/sh
    - -c
    - |
      # Test database connection
      PGPASSWORD=$DB_PASSWORD psql -h $DB_HOST -U $DB_USER -d $DB_NAME -c "SELECT 1"
      
      # Test API endpoint
      curl -f http://{{ include "my-app.fullname" . }}:{{ .Values.service.port }}/health
      
      # Test Redis connection
      redis-cli -h $REDIS_HOST -a $REDIS_PASSWORD ping
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
          name: {{ include "my-app.fullname" . }}-db-secret
          key: password
    - name: REDIS_HOST
      value: {{ .Values.redis.host }}
    - name: REDIS_PASSWORD
      valueFrom:
        secretKeyRef:
          name: {{ include "my-app.fullname" . }}-redis-secret
          key: password
  restartPolicy: Never
```

---

## 10. Best Practices for Advanced Charts

### 10.1 Hooks Best Practices

```
✅ Good:
- Use hooks for setup/teardown
- Set appropriate weights
- Use delete policies
- Make hooks idempotent

❌ Bad:
- Too many hooks
- Hooks that take too long
- Hooks without delete policies
- Non-idempotent hooks
```

### 10.2 Dependencies Best Practices

```
✅ Good:
- Pin dependency versions
- Use Chart.lock
- Document dependencies
- Test with dependencies

❌ Bad:
- Floating versions
- Too many dependencies
- Undocumented dependencies
- Untested combinations
```

### 10.3 Templating Best Practices

```
✅ Good:
- Use helpers for common patterns
- Validate required values
- Use conditionals appropriately
- Comment complex logic

❌ Bad:
- Duplicate template code
- No validation
- Overly complex templates
- No error handling
```

---

## Summary

**Key Takeaways**:
1. ✅ Hooks enable lifecycle management
2. ✅ Dependencies compose complex applications
3. ✅ Advanced templating provides flexibility
4. ✅ Library charts enable reuse
5. ✅ Testing ensures quality

**Next Steps**:
- Part 3: Real-world examples and production best practices

