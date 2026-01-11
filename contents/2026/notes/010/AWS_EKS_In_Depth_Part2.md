# AWS EKS (Elastic Kubernetes Service) - In-Depth Guide

## Part 2: Advanced Topics, Security, Monitoring, and Best Practices

---

## Table of Contents

1. [Security and Access Control](#1-security-and-access-control)
2. [Storage and Persistent Volumes](#2-storage-and-persistent-volumes)
3. [Auto Scaling](#3-auto-scaling)
4. [Monitoring and Observability](#4-monitoring-and-observability)
5. [Logging](#5-logging)
6. [CI/CD Integration](#6-cicd-integration)
7. [Troubleshooting](#7-troubleshooting)
8. [Best Practices](#8-best-practices)
9. [EKS vs Alternatives](#9-eks-vs-alternatives)
10. [Cost Optimization](#10-cost-optimization)

---

## 1. Security and Access Control

### 1.1 IAM Integration

#### Cluster Access Control

EKS uses AWS IAM for cluster authentication.

```bash
# Create IAM user/role for cluster access
aws eks update-kubeconfig \
  --name my-cluster \
  --region us-east-1 \
  --role-arn arn:aws:iam::ACCOUNT_ID:role/EKSAdminRole

# Verify access
kubectl get nodes
```

#### aws-auth ConfigMap

The `aws-auth` ConfigMap maps IAM users/roles to Kubernetes RBAC.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: aws-auth
  namespace: kube-system
data:
  mapRoles: |
    - rolearn: arn:aws:iam::ACCOUNT_ID:role/NodeInstanceRole
      username: system:node:{{EC2PrivateDNSName}}
      groups:
        - system:bootstrappers
        - system:nodes
    - rolearn: arn:aws:iam::ACCOUNT_ID:role/EKSAdminRole
      username: admin
      groups:
        - system:masters
  mapUsers: |
    - userarn: arn:aws:iam::ACCOUNT_ID:user/admin-user
      username: admin-user
      groups:
        - system:masters
```

### 1.2 IRSA (IAM Roles for Service Accounts)

IRSA allows pods to assume IAM roles without storing credentials.

#### Setup OIDC Provider

```bash
# Get cluster OIDC issuer URL
aws eks describe-cluster --name my-cluster --query "cluster.identity.oidc.issuer" --output text

# Create OIDC provider
eksctl utils associate-iam-oidc-provider \
  --cluster my-cluster \
  --approve
```

#### Create IAM Role for Service Account

```bash
# Create IAM policy
cat > s3-read-policy.json <<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket/*",
        "arn:aws:s3:::my-bucket"
      ]
    }
  ]
}
EOF

aws iam create-policy \
  --policy-name S3ReadPolicy \
  --policy-document file://s3-read-policy.json

# Create service account with IRSA
eksctl create iamserviceaccount \
  --cluster my-cluster \
  --name s3-reader-sa \
  --namespace default \
  --attach-policy-arn arn:aws:iam::ACCOUNT_ID:policy/S3ReadPolicy \
  --approve
```

#### Use Service Account in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: s3-reader-pod
  namespace: default
spec:
  serviceAccountName: s3-reader-sa
  containers:
    - name: app
      image: my-app:latest
      # Pod can now access S3 using IAM role
```

### 1.3 Pod Security Policies (PSP)

Pod Security Policies control pod creation and updates.

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted-psp
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: false
```

### 1.4 Network Policies

Network Policies provide pod-level network isolation.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
    - Ingress
  # No ingress rules = deny all ingress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
    - Ingress
  ingress:
    - from:
        - podSelector:
            matchLabels:
              app: frontend
      ports:
        - protocol: TCP
          port: 8080
```

### 1.5 Secrets Management

#### Kubernetes Secrets

```yaml
# Create secret
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: YWRtaW4=  # base64 encoded
  password: cGFzc3dvcmQ=
```

#### AWS Secrets Manager Integration

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: aws-secrets
spec:
  provider: aws
  parameters:
    objects: |
      - objectName: "db-credentials"
        objectType: "secretsmanager"
        jmesPath:
          - path: username
            objectAlias: db-username
          - path: password
            objectAlias: db-password
```

### 1.6 Encryption

#### Encryption at Rest

```bash
# Enable encryption for EKS cluster
aws eks create-cluster \
  --name my-cluster \
  --encryption-config \
    '[{"resources":["secrets"],"provider":{"keyArn":"arn:aws:kms:region:account:key/key-id"}}]' \
  --role-arn arn:aws:iam::ACCOUNT_ID:role/eks-cluster-role
```

#### Encryption in Transit

- **TLS/SSL**: All API server communication uses TLS
- **mTLS**: Service mesh (Istio, Linkerd) for pod-to-pod encryption
- **VPN**: For secure access to cluster from on-premises

---

## 2. Storage and Persistent Volumes

### 2.1 EBS Volumes

EBS provides block storage for persistent volumes.

#### Storage Classes

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ebs-gp3
provisioner: ebs.csi.aws.com
parameters:
  type: gp3
  fsType: ext4
  encrypted: "true"
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```

#### Persistent Volume Claim

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: ebs-gp3
  resources:
    requests:
      storage: 100Gi
```

#### Using PVC in Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mysql-pod
spec:
  containers:
    - name: mysql
      image: mysql:8.0
      volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
  volumes:
    - name: mysql-storage
      persistentVolumeClaim:
        claimName: mysql-pvc
```

### 2.2 EFS Volumes

EFS provides shared file storage (ReadWriteMany).

#### EFS CSI Driver Setup

```bash
# Install EFS CSI driver
kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=release-1.7"

# Create storage class
kubectl apply -f - <<EOF
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: fs-12345678
  directoryPerms: "755"
EOF
```

#### EFS Persistent Volume

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: shared-storage-pvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: efs-sc
  resources:
    requests:
      storage: 5Gi
```

### 2.3 FSx for Lustre

FSx for Lustre provides high-performance file storage.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fsx-lustre-sc
provisioner: fsx.csi.aws.com
parameters:
  subnetId: subnet-12345678
  securityGroupIds: sg-12345678
  deploymentType: PERSISTENT_1
  perUnitStorageThroughput: "200"
```

### 2.4 Volume Snapshots

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: ebs-snapshot-class
driver: ebs.csi.aws.com
deletionPolicy: Retain
---
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: mysql-snapshot
spec:
  volumeSnapshotClassName: ebs-snapshot-class
  source:
    persistentVolumeClaimName: mysql-pvc
```

---

## 3. Auto Scaling

### 3.1 Cluster Autoscaler

Cluster Autoscaler automatically adjusts node group size.

#### Installation

```bash
# Install Cluster Autoscaler
kubectl apply -f https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml

# Edit deployment
kubectl edit deployment cluster-autoscaler -n kube-system

# Add these arguments:
# - --nodes=1:10:my-cluster-nodegroup
# - --balance-similar-node-groups
# - --skip-nodes-with-system-pods=false
```

#### Configuration

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  template:
    spec:
      containers:
        - name: cluster-autoscaler
          image: k8s.gcr.io/autoscaling/cluster-autoscaler:v1.28.0
          command:
            - ./cluster-autoscaler
            - --v=4
            - --stderrthreshold=info
            - --cloud-provider=aws
            - --skip-nodes-with-local-storage=false
            - --expander=least-waste
            - --node-group-auto-discovery=asg:tag=k8s.io/cluster-autoscaler/enabled,k8s.io/cluster-autoscaler/my-cluster
          env:
            - name: AWS_REGION
              value: us-east-1
```

### 3.2 Horizontal Pod Autoscaler (HPA)

HPA automatically scales pods based on metrics.

#### Install Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

#### HPA Example

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
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
          periodSeconds: 15
        - type: Pods
          value: 4
          periodSeconds: 15
      selectPolicy: Max
```

### 3.3 Vertical Pod Autoscaler (VPA)

VPA automatically adjusts pod resource requests and limits.

```bash
# Install VPA
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler/
./hack/vpa-up.sh
```

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: my-app-vpa
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
      - containerName: my-app
        minAllowed:
          cpu: 100m
          memory: 128Mi
        maxAllowed:
          cpu: 2
          memory: 4Gi
```

### 3.4 Karpenter (Advanced Autoscaler)

Karpenter is a node autoscaler that provisions nodes in seconds.

#### Installation

```bash
# Install Karpenter
helm repo add karpenter https://charts.karpenter.sh
helm install karpenter karpenter/karpenter \
  --namespace karpenter \
  --create-namespace \
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=arn:aws:iam::ACCOUNT_ID:role/KarpenterControllerRole
```

#### NodePool Configuration

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    metadata:
      labels:
        intent: apps
    spec:
      requirements:
        - key: kubernetes.io/arch
          operator: In
          values: ["amd64"]
        - key: kubernetes.io/os
          operator: In
          values: ["linux"]
        - key: karpenter.sh/capacity-type
          operator: In
          values: ["spot", "on-demand"]
        - key: node.kubernetes.io/instance-type
          operator: In
          values: ["m5.large", "m5.xlarge", "m5.2xlarge"]
      nodeClassRef:
        name: default
  limits:
    cpu: 1000
  disruption:
    consolidationPolicy: WhenEmpty
    consolidateAfter: 30s
```

---

## 4. Monitoring and Observability

### 4.1 CloudWatch Container Insights

Container Insights provides metrics and logs for EKS clusters.

#### Enable Container Insights

```bash
# Enable for cluster
aws eks update-cluster-config \
  --name my-cluster \
  --logging '{"enable":["api","audit","authenticator","controllerManager","scheduler"]}'

# Install CloudWatch agent
kubectl apply -f https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluentd-quickstart.yaml
```

#### Metrics Available

- **Cluster Metrics**: CPU, memory, network, storage utilization
- **Node Metrics**: Per-node resource usage
- **Pod Metrics**: Per-pod resource consumption
- **Service Metrics**: Request rates, error rates

### 4.2 Prometheus and Grafana

#### Install Prometheus

```bash
# Add Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Install Prometheus
helm install prometheus prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

#### Install Grafana

```bash
# Grafana is included in kube-prometheus-stack
# Access Grafana
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Default credentials: admin/prom-operator
```

### 4.3 AWS X-Ray

X-Ray provides distributed tracing for applications.

#### Install X-Ray Daemon

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: xray-daemon
spec:
  selector:
    matchLabels:
      app: xray-daemon
  template:
    metadata:
      labels:
        app: xray-daemon
    spec:
      containers:
        - name: xray-daemon
          image: amazon/aws-xray-daemon:latest
          ports:
            - containerPort: 2000
              protocol: UDP
```

#### Instrument Application

```java
// Java example
import com.amazonaws.xray.AWSXRay;
import com.amazonaws.xray.AWSXRayRecorderBuilder;

AWSXRayRecorderBuilder builder = AWSXRayRecorderBuilder.standard()
    .withPlugin(new EC2Plugin())
    .withPlugin(new ECSPlugin());
AWSXRay.setGlobalRecorder(builder.build());
```

### 4.4 Custom Metrics

#### Prometheus Adapter

```bash
# Install Prometheus Adapter
helm install prometheus-adapter prometheus-community/prometheus-adapter \
  --namespace monitoring
```

#### Custom Metrics HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: custom-metrics-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Pods
      pods:
        metric:
          name: http_requests_per_second
        target:
          type: AverageValue
          averageValue: "100"
```

---

## 5. Logging

### 5.1 CloudWatch Logs

#### Enable Control Plane Logging

```bash
aws eks update-cluster-config \
  --name my-cluster \
  --logging '{"enable":["api","audit","authenticator","controllerManager","scheduler"]}'
```

#### Fluent Bit for Application Logs

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
  namespace: kube-system
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         1
        Log_Level     info
        Daemon        off
        Parsers_File  parsers.conf

    [INPUT]
        Name              tail
        Path              /var/log/containers/*.log
        Parser            docker
        Tag               kube.*
        Refresh_Interval  5

    [OUTPUT]
        Name  cloudwatch_logs
        Match kube.*
        region us-east-1
        log_group_name /eks/my-cluster
        log_stream_prefix fluent-bit-
        auto_create_group true
```

### 5.2 Centralized Logging

#### Fluentd Configuration

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluent.conf: |
    <source>
      @type tail
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      tag kubernetes.*
      read_from_head true
      <parse>
        @type json
        time_format %Y-%m-%dT%H:%M:%S.%NZ
      </parse>
    </source>

    <match kubernetes.**>
      @type cloudwatch_logs
      log_group_name /eks/my-cluster
      auto_create_stream true
      region us-east-1
    </match>
```

---

## 6. CI/CD Integration

### 6.1 GitOps with ArgoCD

#### Install ArgoCD

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Access ArgoCD UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

#### ArgoCD Application

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/my-app
    targetRevision: main
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

### 6.2 Jenkins on EKS

#### Jenkins Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jenkins
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
        - name: jenkins
          image: jenkins/jenkins:lts
          ports:
            - containerPort: 8080
            - containerPort: 50000
          volumeMounts:
            - name: jenkins-home
              mountPath: /var/jenkins_home
      volumes:
        - name: jenkins-home
          persistentVolumeClaim:
            claimName: jenkins-pvc
```

### 6.3 GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy to EKS

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Configure kubectl
        run: |
          aws eks update-kubeconfig --name my-cluster --region us-east-1
      
      - name: Deploy to EKS
        run: |
          kubectl apply -f k8s/
          kubectl rollout status deployment/my-app
```

### 6.4 GitLab CI/CD

```yaml
# .gitlab-ci.yml
stages:
  - build
  - deploy

build:
  stage: build
  script:
    - docker build -t my-app:$CI_COMMIT_SHA .
    - docker push my-app:$CI_COMMIT_SHA

deploy:
  stage: deploy
  script:
    - kubectl set image deployment/my-app my-app=my-app:$CI_COMMIT_SHA
  only:
    - main
```

---

## 7. Troubleshooting

### 7.1 Common Issues

#### Pods Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name>

# Check events
kubectl get events --sort-by='.lastTimestamp'

# Check logs
kubectl logs <pod-name>

# Common causes:
# - Image pull errors
# - Resource constraints
# - Node capacity
# - Security policies
```

#### Node Not Joining Cluster

```bash
# Check node status
kubectl get nodes

# Check node conditions
kubectl describe node <node-name>

# Verify IAM role
aws iam get-role --role-name NodeInstanceRole

# Check security groups
# Ensure node security group allows communication with control plane
```

#### Network Issues

```bash
# Check service endpoints
kubectl get endpoints

# Test connectivity
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup my-service

# Check network policies
kubectl get networkpolicies

# Verify VPC CNI
kubectl get daemonset aws-node -n kube-system
```

### 7.2 Debugging Tools

#### kubectl debug

```bash
# Create ephemeral debug container
kubectl debug <pod-name> -it --image=busybox --target=<container-name>

# Debug node
kubectl debug node/<node-name> -it --image=busybox
```

#### kubectl exec

```bash
# Execute command in pod
kubectl exec -it <pod-name> -- /bin/sh

# Execute in specific container
kubectl exec -it <pod-name> -c <container-name> -- /bin/sh
```

### 7.3 Log Collection

```bash
# Collect all logs
kubectl logs --all-namespaces --tail=100 > all-logs.txt

# Collect events
kubectl get events --all-namespaces --sort-by='.lastTimestamp' > events.txt

# Describe resources
kubectl describe pod <pod-name> > pod-describe.txt
kubectl describe node <node-name> > node-describe.txt
```

---

## 8. Best Practices

### 8.1 Cluster Design

- **Multi-AZ**: Deploy nodes across multiple Availability Zones
- **Private Subnets**: Use private subnets for worker nodes
- **Separate Node Groups**: Different node groups for different workloads
- **Resource Limits**: Set appropriate resource requests and limits
- **Namespaces**: Use namespaces for resource isolation

### 8.2 Security

- **Least Privilege**: Grant minimum necessary permissions
- **IRSA**: Use IRSA instead of instance profiles for pod-level access
- **Network Policies**: Implement network policies for pod isolation
- **Secrets Management**: Use AWS Secrets Manager or external secrets
- **Image Scanning**: Scan container images for vulnerabilities
- **Pod Security Standards**: Enforce pod security standards

### 8.3 Resource Management

- **Resource Quotas**: Set resource quotas per namespace
- **Limit Ranges**: Define default limits for containers
- **HPA**: Use Horizontal Pod Autoscaler for automatic scaling
- **Cluster Autoscaler**: Enable cluster autoscaler for node scaling
- **Right-Sizing**: Choose appropriate instance types

### 8.4 Monitoring

- **Enable Logging**: Enable all control plane logs
- **Container Insights**: Use CloudWatch Container Insights
- **Prometheus**: Set up Prometheus for metrics
- **Alerts**: Configure alerts for critical metrics
- **Dashboards**: Create dashboards for visibility

### 8.5 Cost Optimization

- **Spot Instances**: Use Spot instances for fault-tolerant workloads
- **Reserved Instances**: Use Reserved Instances for predictable workloads
- **Right-Sizing**: Continuously optimize instance types
- **Cluster Autoscaler**: Scale down unused nodes
- **Resource Limits**: Set appropriate resource limits to avoid over-provisioning

---

## 9. EKS vs Alternatives

### 9.1 EKS vs ECS

| Feature | EKS | ECS |
|---------|-----|-----|
| **Orchestration** | Kubernetes | AWS proprietary |
| **Learning Curve** | Steeper | Easier |
| **Ecosystem** | Large (K8s) | AWS-specific |
| **Control Plane** | Managed | Managed |
| **Flexibility** | High | Medium |
| **Use Case** | K8s workloads | AWS-native apps |

### 9.2 EKS vs Self-Managed K8s

| Feature | EKS | Self-Managed |
|---------|-----|--------------|
| **Setup Time** | Hours | Days/weeks |
| **Maintenance** | Minimal | Ongoing |
| **Control Plane** | AWS manages | You manage |
| **High Availability** | Built-in | You configure |
| **Updates** | AWS handles | Manual |
| **Cost** | Higher | Lower (no EKS fee) |

### 9.3 EKS vs GKE vs AKS

| Feature | EKS | GKE | AKS |
|---------|-----|-----|-----|
| **Cloud Provider** | AWS | GCP | Azure |
| **Integration** | AWS services | GCP services | Azure services |
| **Managed Control Plane** | Yes | Yes | Yes |
| **Fargate Equivalent** | Fargate | GKE Autopilot | ACI |
| **Regional Support** | Global | Global | Global |

---

## 10. Cost Optimization

### 10.1 Instance Selection

- **General Purpose**: t3, m5 for most workloads
- **Compute Optimized**: c5 for CPU-intensive
- **Memory Optimized**: r5 for memory-intensive
- **Spot Instances**: Up to 90% savings for fault-tolerant workloads

### 10.2 Auto Scaling

```yaml
# Cluster Autoscaler configuration
# Scale down aggressively during off-hours
# Use multiple node groups for different instance types
# Enable Spot instances for non-critical workloads
```

### 10.3 Resource Optimization

- **Right-Size Pods**: Set appropriate CPU/memory requests
- **Use HPA**: Scale pods based on demand
- **Cluster Autoscaler**: Scale nodes based on demand
- **Remove Unused Resources**: Clean up unused pods, services, PVCs

### 10.4 Cost Monitoring

```bash
# Enable Cost Explorer
# Tag resources for cost allocation
# Use AWS Cost Anomaly Detection
# Set up billing alerts
```

---

## Summary of Part 2

**Key Takeaways**:
1. **Security**: Use IAM, IRSA, Network Policies, and encryption
2. **Storage**: EBS for block storage, EFS for shared storage
3. **Auto Scaling**: HPA for pods, Cluster Autoscaler/Karpenter for nodes
4. **Monitoring**: CloudWatch, Prometheus, Grafana for observability
5. **CI/CD**: GitOps with ArgoCD, Jenkins, GitHub Actions
6. **Troubleshooting**: Use kubectl debug, describe, logs
7. **Best Practices**: Multi-AZ, least privilege, resource management
8. **Cost Optimization**: Right-sizing, Spot instances, auto scaling

**Complete Guide Summary**:
- **Part 1**: Fundamentals, architecture, setup, networking
- **Part 2**: Security, storage, scaling, monitoring, CI/CD, best practices

This comprehensive guide covers everything needed to understand, deploy, and operate EKS clusters in production environments.

