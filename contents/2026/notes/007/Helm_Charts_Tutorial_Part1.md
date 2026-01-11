# Helm Charts Tutorial - Part 1: Fundamentals and Basics

## Overview

Helm is the package manager for Kubernetes. It simplifies deploying and managing Kubernetes applications by packaging them into reusable units called charts. This tutorial covers Helm fundamentals, installation, and basic chart creation.

---

## 1. Introduction to Helm

### 1.1 What is Helm?

**Helm** is a package manager for Kubernetes, similar to:
- **apt/yum** for Linux
- **npm** for Node.js
- **pip** for Python
- **Maven** for Java

**Key Benefits**:
- **Simplifies Deployment**: Package complex Kubernetes applications
- **Version Management**: Manage application versions
- **Reusability**: Share and reuse charts
- **Configuration Management**: Template-based configuration
- **Dependency Management**: Handle application dependencies

### 1.2 Helm Components

```
┌─────────────────────────────────────────┐
│              HELM ARCHITECTURE          │
└─────────────────────────────────────────┘

1. HELM CLIENT
   - Command-line tool
   - Creates charts
   - Manages releases
   - Communicates with Tiller (Helm 2) or API server (Helm 3)

2. HELM CHARTS
   - Package format
   - Contains templates
   - Includes values files
   - Defines dependencies

3. HELM RELEASES
   - Deployed instances of charts
   - Versioned
   - Can be upgraded/rolled back
```

### 1.3 Helm 2 vs Helm 3

**Helm 2** (Legacy):
- Required Tiller (server-side component)
- Security concerns (Tiller had cluster-admin)
- Being phased out

**Helm 3** (Current):
- No Tiller (client-only)
- Better security
- Improved upgrade mechanism
- Better dependency management
- **We'll use Helm 3 in this tutorial**

### 1.4 Key Concepts

**Chart**: A package of pre-configured Kubernetes resources
**Release**: An instance of a chart running in a Kubernetes cluster
**Repository**: A place where charts are stored and shared
**Template**: YAML files with placeholders for values
**Values**: Configuration parameters for templates

---

## 2. Installation

### 2.1 Prerequisites

- Kubernetes cluster (local or remote)
- `kubectl` configured
- Docker (for building images)

### 2.2 Install Helm 3

#### macOS

```bash
# Using Homebrew
brew install helm

# Or using script
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

#### Linux

```bash
# Download and install
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Or using package manager
# Debian/Ubuntu
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

#### Windows

```powershell
# Using Chocolatey
choco install kubernetes-helm

# Or using Scoop
scoop install helm
```

### 2.3 Verify Installation

```bash
# Check Helm version
helm version

# Output:
# version.BuildInfo{Version:"v3.12.0", GitCommit:"...", GitTreeState:"clean", GoVersion:"go1.20.5"}
```

### 2.4 Initialize Helm

```bash
# Helm 3 doesn't need initialization
# Just verify kubectl access
kubectl cluster-info

# Add a chart repository
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

---

## 3. Helm Basics

### 3.1 Basic Commands

```bash
# Search for charts
helm search repo nginx

# Install a chart
helm install my-nginx stable/nginx

# List installed releases
helm list

# Get release information
helm status my-nginx

# Uninstall a release
helm uninstall my-nginx
```

### 3.2 Chart Repository

```bash
# Add repository
helm repo add bitnami https://charts.bitnami.com/bitnami

# List repositories
helm repo list

# Update repository
helm repo update

# Remove repository
helm repo remove bitnami
```

### 3.3 Release Management

```bash
# Install with custom name
helm install my-release stable/nginx

# Install in specific namespace
helm install my-release stable/nginx --namespace production --create-namespace

# Install with custom values
helm install my-release stable/nginx -f my-values.yaml

# Upgrade release
helm upgrade my-release stable/nginx

# Rollback release
helm rollback my-release 1

# View release history
helm history my-release
```

---

## 4. Chart Structure

### 4.1 Basic Chart Structure

```
my-chart/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
├── templates/          # Template files
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
└── charts/            # Chart dependencies (optional)
```

### 4.2 Chart.yaml

**Purpose**: Defines chart metadata

**Example**:

```yaml
apiVersion: v2
name: my-webapp
description: A Helm chart for my web application
type: application
version: 1.0.0
appVersion: "1.0.0"
keywords:
  - webapp
  - nginx
home: https://example.com
sources:
  - https://github.com/example/my-webapp
maintainers:
  - name: John Doe
    email: john@example.com
```

**Key Fields**:
- `apiVersion`: Chart API version (v2 for Helm 3)
- `name`: Chart name
- `version`: Chart version (semantic versioning)
- `appVersion`: Application version
- `type`: `application` or `library`

### 4.3 values.yaml

**Purpose**: Default configuration values

**Example**:

```yaml
# Default values for my-webapp
replicaCount: 3

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21"

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: ""
  annotations: {}
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

---

## 5. Creating Your First Chart

### 5.1 Create Chart Scaffold

```bash
# Create a new chart
helm create my-webapp

# This creates:
# my-webapp/
# ├── Chart.yaml
# ├── values.yaml
# ├── .helmignore
# └── templates/
#     ├── deployment.yaml
#     ├── service.yaml
#     ├── ingress.yaml
#     ├── hpa.yaml
#     ├── serviceaccount.yaml
#     ├── _helpers.tpl
#     └── tests/
#         └── test-connection.yaml
```

### 5.2 Simple Chart Example

Let's create a simple Nginx web application chart:

#### Step 1: Create Chart Structure

```bash
mkdir -p my-nginx/templates
cd my-nginx
```

#### Step 2: Create Chart.yaml

```yaml
apiVersion: v2
name: my-nginx
description: A simple Nginx Helm chart
type: application
version: 0.1.0
appVersion: "1.21"
```

#### Step 3: Create values.yaml

```yaml
replicaCount: 2

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21"

service:
  type: ClusterIP
  port: 80
  targetPort: 80

resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

#### Step 4: Create templates/deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-nginx.fullname" . }}
  labels:
    {{- include "my-nginx.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-nginx.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-nginx.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
```

#### Step 5: Create templates/service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-nginx.fullname" . }}
  labels:
    {{- include "my-nginx.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "my-nginx.selectorLabels" . | nindent 4 }}
```

#### Step 6: Create templates/_helpers.tpl

```yaml
{{/*
Common labels
*/}}
{{- define "my-nginx.labels" -}}
helm.sh/chart: {{ include "my-nginx.chart" . }}
{{ include "my-nginx.selectorLabels" . }}
{{- if .Chart.AppVersion }}
app.kubernetes.io/version: {{ .Chart.AppVersion | quote }}
{{- end }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
{{- end }}

{{/*
Selector labels
*/}}
{{- define "my-nginx.selectorLabels" -}}
app.kubernetes.io/name: {{ include "my-nginx.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end }}

{{/*
Chart name and version
*/}}
{{- define "my-nginx.chart" -}}
{{- printf "%s-%s" .Chart.Name .Chart.Version | replace "+" "_" | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Common name
*/}}
{{- define "my-nginx.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Full name
*/}}
{{- define "my-nginx.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}
```

### 5.3 Validate Chart

```bash
# Lint the chart
helm lint my-nginx

# Dry-run to see rendered templates
helm install my-nginx . --dry-run --debug

# Template rendering (without installing)
helm template my-nginx .
```

### 5.4 Install Chart

```bash
# Install the chart
helm install my-nginx .

# Check status
helm status my-nginx

# List releases
helm list

# View rendered templates
kubectl get all -l app.kubernetes.io/instance=my-nginx
```

---

## 6. Template Syntax

### 6.1 Basic Template Syntax

Helm uses Go templates with Sprig functions.

#### Variables

```yaml
# Access values
{{ .Values.replicaCount }}

# Access release information
{{ .Release.Name }}
{{ .Release.Namespace }}

# Access chart information
{{ .Chart.Name }}
{{ .Chart.Version }}
```

#### Conditionals

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "my-nginx.fullname" . }}
{{- end }}
```

#### Loops

```yaml
{{- range .Values.env }}
- name: {{ .name }}
  value: {{ .value }}
{{- end }}
```

#### Functions

```yaml
# String functions
{{ .Values.name | upper }}
{{ .Values.name | lower }}
{{ .Values.name | title }}

# Default values
{{ .Values.image.tag | default "latest" }}

# Required values
{{ required "image.repository is required" .Values.image.repository }}

# Include templates
{{ include "my-nginx.fullname" . }}
```

### 6.2 Common Template Patterns

#### Pattern 1: Conditional Resource Creation

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "my-nginx.fullname" . }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "my-nginx.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

#### Pattern 2: Environment Variables

```yaml
env:
{{- range $key, $value := .Values.env }}
- name: {{ $key }}
  value: {{ $value | quote }}
{{- end }}

# Or from a list
env:
{{- range .Values.envList }}
- name: {{ .name }}
  value: {{ .value | quote }}
{{- end }}
```

#### Pattern 3: Annotations

```yaml
metadata:
  annotations:
    {{- range $key, $value := .Values.annotations }}
    {{ $key }}: {{ $value | quote }}
    {{- end }}
```

---

## 7. Docker Integration

### 7.1 Building Docker Images

Create a Dockerfile for your application:

```dockerfile
# Dockerfile
FROM nginx:1.21-alpine

# Copy custom configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Copy application files
COPY html/ /usr/share/nginx/html/

# Expose port
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

### 7.2 Building and Pushing Images

```bash
# Build image
docker build -t my-registry/my-webapp:1.0.0 .

# Tag for registry
docker tag my-webapp:1.0.0 my-registry/my-webapp:1.0.0

# Push to registry
docker push my-registry/my-webapp:1.0.0
```

### 7.3 Using Custom Images in Helm

Update `values.yaml`:

```yaml
image:
  repository: my-registry/my-webapp
  pullPolicy: Always
  tag: "1.0.0"
```

Update `deployment.yaml`:

```yaml
containers:
- name: webapp
  image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
  imagePullPolicy: {{ .Values.image.pullPolicy }}
```

### 7.4 Image Pull Secrets

If using private registry:

```yaml
# values.yaml
image:
  repository: my-private-registry/my-webapp
  pullPolicy: Always
  tag: "1.0.0"
  pullSecrets:
    - name: registry-secret
```

```yaml
# templates/deployment.yaml
spec:
  {{- if .Values.image.pullSecrets }}
  imagePullSecrets:
    {{- range .Values.image.pullSecrets }}
    - name: {{ . }}
    {{- end }}
  {{- end }}
  containers:
  - name: webapp
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
```

Create secret:

```bash
kubectl create secret docker-registry registry-secret \
  --docker-server=my-private-registry \
  --docker-username=myuser \
  --docker-password=mypassword \
  --docker-email=myemail@example.com
```

---

## 8. Kubernetes Deployment Example

### 8.1 Complete Deployment Example

Let's create a complete web application with:
- Deployment
- Service
- Ingress
- ConfigMap
- Secret

#### values.yaml

```yaml
replicaCount: 3

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: "1.21"

service:
  type: ClusterIP
  port: 80
  targetPort: 80

ingress:
  enabled: true
  className: "nginx"
  annotations:
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
  hosts:
    - host: myapp.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: myapp-tls
      hosts:
        - myapp.example.com

configMap:
  enabled: true
  data:
    nginx.conf: |
      user nginx;
      worker_processes auto;
      events {
        worker_connections 1024;
      }
      http {
        include /etc/nginx/mime.types;
        default_type application/octet-stream;
        sendfile on;
        keepalive_timeout 65;
        server {
          listen 80;
          location / {
            root /usr/share/nginx/html;
            index index.html;
          }
        }
      }

secret:
  enabled: true
  data:
    api-key: "base64-encoded-api-key"

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 3
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

#### templates/configmap.yaml

```yaml
{{- if .Values.configMap.enabled }}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "my-nginx.fullname" . }}-config
  labels:
    {{- include "my-nginx.labels" . | nindent 4 }}
data:
  {{- range $key, $value := .Values.configMap.data }}
  {{ $key }}: |
{{ $value | indent 4 }}
  {{- end }}
{{- end }}
```

#### templates/secret.yaml

```yaml
{{- if .Values.secret.enabled }}
apiVersion: v1
kind: Secret
metadata:
  name: {{ include "my-nginx.fullname" . }}-secret
  labels:
    {{- include "my-nginx.labels" . | nindent 4 }}
type: Opaque
data:
  {{- range $key, $value := .Values.secret.data }}
  {{ $key }}: {{ $value | b64enc }}
  {{- end }}
{{- end }}
```

#### templates/deployment.yaml (Updated)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-nginx.fullname" . }}
  labels:
    {{- include "my-nginx.labels" . | nindent 4 }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "my-nginx.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "my-nginx.selectorLabels" . | nindent 8 }}
    spec:
      {{- if .Values.image.pullSecrets }}
      imagePullSecrets:
        {{- range .Values.image.pullSecrets }}
        - name: {{ . }}
        {{- end }}
      {{- end }}
      containers:
      - name: nginx
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
          protocol: TCP
        {{- if .Values.configMap.enabled }}
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
        {{- end }}
        {{- if .Values.secret.enabled }}
        env:
        - name: API_KEY
          valueFrom:
            secretKeyRef:
              name: {{ include "my-nginx.fullname" . }}-secret
              key: api-key
        {{- end }}
        resources:
          {{- toYaml .Values.resources | nindent 10 }}
      {{- if .Values.configMap.enabled }}
      volumes:
      - name: config
        configMap:
          name: {{ include "my-nginx.fullname" . }}-config
      {{- end }}
```

#### templates/hpa.yaml

```yaml
{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "my-nginx.fullname" . }}
  labels:
    {{- include "my-nginx.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "my-nginx.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
{{- end }}
```

### 8.2 Deploy to Kubernetes

```bash
# Validate chart
helm lint my-nginx

# Dry-run
helm install my-nginx . --dry-run --debug

# Install
helm install my-nginx . --namespace production --create-namespace

# Check deployment
kubectl get all -n production

# View logs
kubectl logs -f deployment/my-nginx -n production

# Upgrade
helm upgrade my-nginx . --set replicaCount=5

# Rollback
helm rollback my-nginx 1
```

---

## 9. Testing Charts

### 9.1 Chart Testing

```bash
# Lint chart
helm lint my-nginx

# Template rendering
helm template my-nginx .

# Dry-run installation
helm install my-nginx . --dry-run --debug

# Validate against Kubernetes API
helm install my-nginx . --dry-run --debug --validate
```

### 9.2 Test Templates

Create `templates/tests/test-connection.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "my-nginx.fullname" . }}-test-connection"
  labels:
    {{- include "my-nginx.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test
spec:
  containers:
  - name: wget
    image: busybox
    command: ['wget']
    args: ['{{ include "my-nginx.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never
```

Run tests:

```bash
helm test my-nginx
```

---

## 10. Best Practices

### 10.1 Chart Organization

```
✅ Good:
- Clear structure
- Logical grouping
- Consistent naming
- Documentation

❌ Bad:
- Flat structure
- Inconsistent naming
- No documentation
```

### 10.2 Values Management

```
✅ Good:
- Sensible defaults
- Well-documented values
- Type validation
- Grouped logically

❌ Bad:
- No defaults
- Undocumented
- Mixed types
- Flat structure
```

### 10.3 Template Best Practices

```
✅ Good:
- Use helpers for common patterns
- Validate required values
- Use conditionals appropriately
- Comment complex logic

❌ Bad:
- Duplicate code
- No validation
- Complex nested conditionals
- No comments
```

---

## Summary

**Key Takeaways**:
1. ✅ Helm simplifies Kubernetes deployments
2. ✅ Charts package Kubernetes resources
3. ✅ Templates use Go template syntax
4. ✅ Values provide configuration
5. ✅ Docker images integrate seamlessly
6. ✅ Testing ensures chart quality

**Next Steps**:
- Part 2: Advanced features (hooks, dependencies, subcharts)
- Part 3: Real-world examples and best practices

