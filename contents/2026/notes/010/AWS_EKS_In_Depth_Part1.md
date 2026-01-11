# AWS EKS (Elastic Kubernetes Service) - In-Depth Guide

## Part 1: Fundamentals, Architecture, and Core Concepts

---

## Table of Contents

1. [Introduction to EKS](#1-introduction-to-eks)
2. [What is Kubernetes?](#2-what-is-kubernetes)
3. [Why Use EKS?](#3-why-use-eks)
4. [EKS Architecture](#4-eks-architecture)
5. [Core Components](#5-core-components)
6. [Cluster Setup and Configuration](#6-cluster-setup-and-configuration)
7. [Node Groups and Compute](#7-node-groups-and-compute)
8. [Networking Fundamentals](#8-networking-fundamentals)

---

## 1. Introduction to EKS

### 1.1 What is EKS?

**AWS Elastic Kubernetes Service (EKS)** is a fully managed Kubernetes service that makes it easy to run Kubernetes on AWS without needing to install, operate, and maintain your own Kubernetes control plane.

### 1.2 Key Characteristics

- **Managed Control Plane**: AWS manages the Kubernetes control plane (API server, etcd, scheduler, controller manager)
- **Kubernetes Compatible**: Runs standard Kubernetes, compatible with existing Kubernetes tools and plugins
- **High Availability**: Control plane runs across multiple Availability Zones
- **Security**: Integrated with AWS IAM, VPC, and security services
- **Scalable**: Automatically scales control plane based on load

### 1.3 EKS vs Self-Managed Kubernetes

| Aspect | Self-Managed K8s | EKS |
|--------|------------------|-----|
| **Control Plane** | You manage | AWS manages |
| **Setup Time** | Days/weeks | Hours |
| **Maintenance** | Ongoing | Minimal |
| **High Availability** | You configure | Built-in |
| **Updates** | Manual | AWS handles |
| **Cost** | EC2 + management | EC2 + EKS fee |
| **Compatibility** | Standard K8s | Standard K8s |

---

## 2. What is Kubernetes?

### 2.1 Kubernetes Overview

Kubernetes (K8s) is an open-source container orchestration platform that automates deployment, scaling, and management of containerized applications.

### 2.2 Core Kubernetes Concepts

#### Pod
- Smallest deployable unit in Kubernetes
- Contains one or more containers
- Shares network and storage resources
- Ephemeral (can be created/destroyed)

#### Deployment
- Manages replica sets of pods
- Ensures desired number of pods are running
- Handles rolling updates and rollbacks
- Declarative configuration

#### Service
- Stable network endpoint for pods
- Load balances traffic to pods
- Provides service discovery
- Types: ClusterIP, NodePort, LoadBalancer, ExternalName

#### Namespace
- Virtual cluster within a physical cluster
- Resource isolation and organization
- Default namespaces: default, kube-system, kube-public

#### ConfigMap & Secret
- ConfigMap: Non-sensitive configuration data
- Secret: Sensitive data (passwords, tokens)
- Mounted as volumes or environment variables

### 2.3 Kubernetes Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Kubernetes Cluster                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │           Control Plane (Master Nodes)            │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │  │
│  │  │ API      │  │ etcd     │  │ Scheduler│       │  │
│  │  │ Server   │  │ (State)  │  │          │       │  │
│  │  └──────────┘  └──────────┘  └──────────┘       │  │
│  │  ┌──────────┐  ┌──────────┐                      │  │
│  │  │ Controller│  │ Cloud    │                      │  │
│  │  │ Manager  │  │ Controller│                      │  │
│  │  └──────────┘  └──────────┘                      │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Worker Nodes (EC2 Instances)         │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │  │
│  │  │ kubelet  │  │ kube-    │  │ Container│       │  │
│  │  │          │  │ proxy     │  │ Runtime  │       │  │
│  │  └──────────┘  └──────────┘  └──────────┘       │  │
│  │                                                  │  │
│  │  ┌──────────────────────────────────────────┐   │  │
│  │  │ Pods (Containers)                        │   │  │
│  │  │ ┌──────┐ ┌──────┐ ┌──────┐              │   │  │
│  │  │ │ App  │ │ App  │ │ App  │              │   │  │
│  │  │ └──────┘ └──────┘ └──────┘              │   │  │
│  │  └──────────────────────────────────────────┘   │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Why Use EKS?

### 3.1 Benefits of EKS

#### Managed Control Plane
- **No Setup**: AWS provisions and manages the control plane
- **High Availability**: Runs across multiple AZs automatically
- **Automatic Updates**: AWS handles Kubernetes version updates
- **Monitoring**: CloudWatch integration for control plane metrics
- **Security**: AWS handles security patches and compliance

#### AWS Integration
- **IAM Integration**: Use AWS IAM for authentication and authorization
- **VPC Networking**: Native VPC integration for network isolation
- **Load Balancers**: Integration with ALB and NLB
- **Storage**: EBS volumes for persistent storage
- **Secrets**: Integration with AWS Secrets Manager

#### Production Ready
- **Compliance**: SOC, PCI-DSS, HIPAA eligible
- **Scalability**: Handles thousands of nodes
- **Reliability**: 99.95% SLA for control plane
- **Support**: Enterprise support available

### 3.2 Use Cases

- **Microservices**: Deploy and manage microservices architectures
- **CI/CD Pipelines**: Container-based continuous deployment
- **Hybrid Cloud**: Connect on-premises K8s with EKS
- **Machine Learning**: Run ML workloads with GPU support
- **Batch Processing**: Large-scale batch jobs
- **Web Applications**: Scalable web applications

### 3.3 When to Choose EKS

**Choose EKS when**:
- ✅ You need Kubernetes compatibility
- ✅ You want managed control plane
- ✅ You need AWS service integration
- ✅ You have Kubernetes expertise
- ✅ You need production-grade reliability

**Consider alternatives when**:
- ❌ Simple applications (ECS might be simpler)
- ❌ No Kubernetes expertise (ECS Fargate)
- ❌ Cost-sensitive (self-managed might be cheaper)
- ❌ Very small scale (ECS might be sufficient)

---

## 4. EKS Architecture

### 4.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        AWS Account                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              EKS Control Plane (Managed)              │  │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐     │  │
│  │  │ AZ-1       │  │ AZ-2       │  │ AZ-3       │     │  │
│  │  │ API Server │  │ API Server │  │ API Server │     │  │
│  │  │ etcd       │  │ etcd       │  │ etcd       │     │  │
│  │  └────────────┘  └────────────┘  └────────────┘     │  │
│  └──────────────────────────────────────────────────────┘  │
│                          │                                   │
│                          │ VPC Endpoint                     │
│                          ▼                                   │
│  ┌──────────────────────────────────────────────────────┐  │
│  │                    VPC (Your Account)                  │  │
│  │                                                         │  │
│  │  ┌──────────────────────────────────────────────┐    │  │
│  │  │         Private Subnet (AZ-1)                 │    │  │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐   │    │  │
│  │  │  │ Node     │  │ Node     │  │ Node     │   │    │  │
│  │  │  │ Group 1  │  │ Group 1  │  │ Group 1  │   │    │  │
│  │  │  └──────────┘  └──────────┘  └──────────┘   │    │  │
│  │  └──────────────────────────────────────────────┘    │  │
│  │                                                         │  │
│  │  ┌──────────────────────────────────────────────┐    │  │
│  │  │         Private Subnet (AZ-2)                 │    │  │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐   │    │  │
│  │  │  │ Node     │  │ Node     │  │ Node     │   │    │  │
│  │  │  │ Group 2  │  │ Group 2  │  │ Group 2  │   │    │  │
│  │  │  └──────────┘  └──────────┘  └──────────┘   │    │  │
│  │  └──────────────────────────────────────────────┘    │  │
│  │                                                         │  │
│  │  ┌──────────────────────────────────────────────┐    │  │
│  │  │              Public Subnet                     │    │  │
│  │  │  ┌──────────┐                                 │    │  │
│  │  │  │ ALB/NLB  │                                 │    │  │
│  │  │  └──────────┘                                 │    │  │
│  │  └──────────────────────────────────────────────┘    │  │
│  │                                                         │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 4.2 Control Plane Architecture

The EKS control plane consists of:

#### API Server
- Entry point for all Kubernetes operations
- Validates and processes API requests
- Communicates with etcd for state
- Handles authentication and authorization

#### etcd
- Distributed key-value store
- Stores cluster state and configuration
- Highly available across multiple AZs
- AWS manages backups and recovery

#### Scheduler
- Assigns pods to nodes based on resource requirements
- Considers node capacity, affinity rules, and constraints
- Handles pod placement decisions

#### Controller Manager
- Runs controllers that maintain desired state
- Replication Controller, Deployment Controller, etc.
- Continuously reconciles actual vs desired state

#### Cloud Controller Manager
- AWS-specific controllers
- Integrates with AWS services (ELB, EBS, etc.)
- Manages AWS resources for Kubernetes objects

### 4.3 Data Plane Architecture

The data plane consists of:

#### Worker Nodes (EC2 Instances)
- Run containerized applications
- Managed by you (or AWS Fargate)
- Can be in Auto Scaling Groups
- Multiple instance types supported

#### kubelet
- Agent running on each node
- Communicates with API server
- Manages pods and containers
- Reports node status

#### kube-proxy
- Network proxy on each node
- Implements Service abstraction
- Handles load balancing
- Manages iptables rules

#### Container Runtime
- Runs containers (Docker, containerd, CRI-O)
- Pulls container images
- Manages container lifecycle

---

## 5. Core Components

### 5.1 EKS Cluster

An EKS cluster is the foundation that consists of:
- **Control Plane**: Managed by AWS
- **VPC**: Your VPC where nodes run
- **IAM Roles**: For cluster and node authentication
- **Security Groups**: Network security rules
- **CloudWatch Logs**: Control plane logging

### 5.2 Node Groups

Node groups are collections of EC2 instances that run your workloads.

#### Managed Node Groups
- **AWS Managed**: AWS handles node lifecycle
- **Auto Scaling**: Integrated with Auto Scaling Groups
- **Updates**: Rolling updates handled automatically
- **Simpler**: Less configuration needed

#### Self-Managed Node Groups
- **You Manage**: Full control over nodes
- **Flexibility**: Custom AMIs, configurations
- **Complexity**: More setup and maintenance
- **Use Case**: Special requirements or custom needs

#### Fargate
- **Serverless**: No node management
- **Pay per Pod**: Only pay for running pods
- **Isolation**: Each pod gets dedicated resources
- **Limitations**: No DaemonSets, privileged containers

### 5.3 Add-ons

EKS supports add-ons for extended functionality:

#### Core Add-ons
- **VPC CNI**: Networking plugin for pod networking
- **kube-proxy**: Network proxy for Services
- **CoreDNS**: DNS server for service discovery
- **EBS CSI Driver**: Persistent volume support

#### Optional Add-ons
- **AWS Load Balancer Controller**: ALB/NLB integration
- **Cluster Autoscaler**: Automatic node scaling
- **Metrics Server**: Resource metrics for HPA
- **EFS CSI Driver**: EFS volume support

### 5.4 IAM Roles

#### Cluster Service Role
- Used by EKS control plane
- Permissions for creating resources (load balancers, etc.)
- Created during cluster creation

#### Node Group Role
- Attached to EC2 instances in node groups
- Permissions for:
  - Pulling container images from ECR
  - Writing logs to CloudWatch
  - Accessing other AWS services

#### IRSA (IAM Roles for Service Accounts)
- Allows pods to assume IAM roles
- Uses OIDC provider
- Fine-grained permissions per service account
- Best practice for pod-level AWS access

---

## 6. Cluster Setup and Configuration

### 6.1 Prerequisites

Before creating an EKS cluster, you need:

- **AWS Account**: With appropriate permissions
- **VPC**: With subnets in at least 2 AZs
- **IAM Roles**: Cluster and node group roles
- **kubectl**: Kubernetes command-line tool
- **eksctl or AWS CLI**: For cluster creation
- **AWS CLI**: Configured with credentials

### 6.2 Creating Cluster with eksctl

**eksctl** is the official CLI tool for EKS:

```bash
# Install eksctl
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin

# Create cluster (simple)
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --nodegroup-name standard-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4

# Create cluster (advanced)
eksctl create cluster \
  --name production-cluster \
  --region us-east-1 \
  --version 1.28 \
  --vpc-private-subnets subnet-123,subnet-456 \
  --vpc-public-subnets subnet-789,subnet-012 \
  --nodegroup-name workers \
  --node-type m5.large \
  --nodes 3 \
  --nodes-min 2 \
  --nodes-max 10 \
  --managed \
  --ssh-access \
  --ssh-public-key ~/.ssh/id_rsa.pub
```

### 6.3 Creating Cluster with AWS CLI

```bash
# Create IAM role for cluster
aws iam create-role \
  --role-name eks-cluster-role \
  --assume-role-policy-document file://cluster-role-trust-policy.json

# Attach policy
aws iam attach-role-policy \
  --role-name eks-cluster-role \
  --policy-arn arn:aws:iam::aws:policy/AmazonEKSClusterPolicy

# Create cluster
aws eks create-cluster \
  --name my-cluster \
  --role-arn arn:aws:iam::ACCOUNT_ID:role/eks-cluster-role \
  --resources-vpc-config subnetIds=subnet-123,subnet-456,securityGroupIds=sg-789 \
  --version 1.28

# Wait for cluster to be active
aws eks wait cluster-active --name my-cluster

# Configure kubectl
aws eks update-kubeconfig --name my-cluster --region us-east-1
```

### 6.4 Cluster Configuration File (eksctl)

```yaml
# cluster-config.yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: production-cluster
  region: us-east-1
  version: "1.28"

vpc:
  id: vpc-12345678
  subnets:
    private:
      us-east-1a: { id: subnet-private1a }
      us-east-1b: { id: subnet-private1b }
    public:
      us-east-1a: { id: subnet-public1a }
      us-east-1b: { id: subnet-public1b }

iam:
  withOIDC: true
  serviceAccounts:
    - metadata:
        name: aws-load-balancer-controller
        namespace: kube-system
      wellKnownPolicies:
        awsLoadBalancerController: true

managedNodeGroups:
  - name: standard-workers
    instanceType: m5.large
    minSize: 2
    maxSize: 10
    desiredCapacity: 3
    volumeSize: 50
    ssh:
      allow: true
      publicKeyPath: ~/.ssh/id_rsa.pub
    labels:
      role: worker
    tags:
      Environment: production

addons:
  - name: vpc-cni
    version: latest
  - name: coredns
    version: latest
  - name: kube-proxy
    version: latest
  - name: aws-ebs-csi-driver
    version: latest
```

Create cluster:
```bash
eksctl create cluster -f cluster-config.yaml
```

### 6.5 Verifying Cluster

```bash
# Check cluster status
aws eks describe-cluster --name my-cluster

# Verify kubectl connection
kubectl cluster-info

# Get nodes
kubectl get nodes

# Get all resources
kubectl get all --all-namespaces
```

---

## 7. Node Groups and Compute

### 7.1 Managed Node Groups

Managed node groups are the recommended way to run worker nodes.

#### Features
- **Automatic Updates**: AWS handles Kubernetes and OS updates
- **Auto Scaling**: Integrated with Auto Scaling Groups
- **Health Monitoring**: Automatic node health checks
- **Simplified Management**: Less operational overhead

#### Creating Managed Node Group

```bash
# Using eksctl
eksctl create nodegroup \
  --cluster my-cluster \
  --name managed-workers \
  --node-type t3.medium \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 5 \
  --managed

# Using AWS CLI
aws eks create-nodegroup \
  --cluster-name my-cluster \
  --nodegroup-name managed-workers \
  --node-role arn:aws:iam::ACCOUNT_ID:role/NodeInstanceRole \
  --subnets subnet-123 subnet-456 \
  --instance-types t3.medium \
  --scaling-config minSize=1,maxSize=5,desiredSize=3 \
  --ami-type AL2_x86_64
```

### 7.2 Self-Managed Node Groups

Self-managed node groups give you full control.

#### Use Cases
- Custom AMIs with pre-installed software
- Special instance types or configurations
- Custom bootstrap scripts
- GPU instances with specific drivers

#### Creating Self-Managed Node Group

```bash
# Using eksctl
eksctl create nodegroup \
  --cluster my-cluster \
  --name self-managed-workers \
  --node-type g4dn.xlarge \
  --nodes 2 \
  --managed=false \
  --ssh-access

# Using CloudFormation or Terraform
# Requires more configuration
```

### 7.3 Fargate

Fargate provides serverless compute for pods.

#### Features
- **No Node Management**: AWS manages infrastructure
- **Pay per Pod**: Only pay for running pods
- **Isolation**: Each pod gets dedicated resources
- **Automatic Scaling**: Scales with pod demand

#### Limitations
- No DaemonSets
- No privileged containers
- No host networking
- No host volumes
- Specific storage limitations

#### Creating Fargate Profile

```bash
# Create Fargate profile
aws eks create-fargate-profile \
  --cluster-name my-cluster \
  --fargate-profile-name my-fargate-profile \
  --pod-execution-role-arn arn:aws:iam::ACCOUNT_ID:role/eks-fargate-profile-role \
  --subnets subnet-123 subnet-456 \
  --selectors namespace=default,namespace=kube-system
```

### 7.4 Node Group Configuration

#### Instance Types

**General Purpose**:
- t3, t3a: Burstable performance
- m5, m5a: Balanced compute, memory, network

**Compute Optimized**:
- c5, c5a: High-performance compute

**Memory Optimized**:
- r5, r5a: High memory-to-CPU ratio

**GPU Instances**:
- g4dn: NVIDIA T4 GPUs
- p3, p4: High-performance GPUs

#### Auto Scaling Configuration

```yaml
# Node group auto scaling
scalingConfig:
  minSize: 2        # Minimum nodes
  maxSize: 10       # Maximum nodes
  desiredSize: 3    # Desired nodes
```

#### Update Configuration

```bash
# Update node group
aws eks update-nodegroup-config \
  --cluster-name my-cluster \
  --nodegroup-name my-nodegroup \
  --scaling-config minSize=2,maxSize=15,desiredSize=5

# Update node group version
aws eks update-nodegroup-version \
  --cluster-name my-cluster \
  --nodegroup-name my-nodegroup \
  --kubernetes-version 1.28
```

---

## 8. Networking Fundamentals

### 8.1 VPC Networking

EKS clusters run in your VPC, providing network isolation and control.

#### Subnet Requirements
- **Minimum 2 Subnets**: In different Availability Zones
- **Private Subnets**: Recommended for worker nodes
- **Public Subnets**: For load balancers, NAT gateways
- **CIDR Blocks**: Sufficient IP addresses for pods

#### IP Address Planning

```
VPC: 10.0.0.0/16 (65,536 IPs)

Subnets:
- Public Subnet 1: 10.0.1.0/24 (256 IPs)
- Public Subnet 2: 10.0.2.0/24 (256 IPs)
- Private Subnet 1: 10.0.10.0/24 (256 IPs)
- Private Subnet 2: 10.0.11.0/24 (256 IPs)

Pod IPs (VPC CNI):
- Pod CIDR per node: /24 (256 IPs per node)
- Total pod capacity: Depends on node count
```

### 8.2 VPC CNI (Container Network Interface)

VPC CNI is the networking plugin for EKS pods.

#### Features
- **Native VPC Integration**: Pods get VPC IP addresses
- **Security Groups**: Can attach security groups to pods
- **Network Policies**: Support for Kubernetes Network Policies
- **IP Address Management**: Efficient IP allocation

#### Configuration

```bash
# Install VPC CNI add-on
aws eks create-addon \
  --cluster-name my-cluster \
  --addon-name vpc-cni \
  --addon-version v1.14.0-eksbuild.1

# Configure VPC CNI
kubectl set env daemonset aws-node -n kube-system \
  ENABLE_PREFIX_DELEGATION=true \
  WARM_PREFIX_TARGET=1
```

### 8.3 Service Networking

#### ClusterIP (Default)
- Internal cluster IP
- Accessible only within cluster
- Used for inter-pod communication

#### NodePort
- Exposes service on each node's IP at a static port
- Accessible from outside cluster
- Port range: 30000-32767

#### LoadBalancer
- Creates AWS Load Balancer (ALB or NLB)
- External IP for service
- Integrated with AWS Load Balancer Controller

#### ExternalName
- Maps service to external DNS name
- No proxy, just DNS CNAME

### 8.4 Ingress

Ingress provides HTTP/HTTPS routing to services.

#### AWS Load Balancer Controller

```bash
# Install AWS Load Balancer Controller
kubectl apply -k "github.com/aws/eks-charts/aws-load-balancer-controller//crds?ref=master"

helm repo add eks https://aws.github.io/eks-charts
helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=my-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller
```

#### Ingress Example

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-app-ingress
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
spec:
  rules:
    - host: myapp.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-app-service
                port:
                  number: 80
```

### 8.5 Network Policies

Network Policies provide pod-level network isolation.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
  namespace: production
spec:
  podSelector:
    matchLabels:
      app: frontend
  policyTypes:
    - Egress
  egress:
    - to:
        - podSelector:
            matchLabels:
              app: backend
      ports:
        - protocol: TCP
          port: 8080
```

---

## Summary of Part 1

**Key Takeaways**:
1. **EKS** is managed Kubernetes service on AWS
2. **Control Plane** is fully managed by AWS across multiple AZs
3. **Node Groups** can be managed, self-managed, or Fargate
4. **Networking** uses VPC CNI for native VPC integration
5. **Setup** can be done with eksctl, AWS CLI, or CloudFormation
6. **Add-ons** extend functionality (VPC CNI, CoreDNS, etc.)
7. **IAM Integration** provides security and access control

**Next**: Part 2 will cover advanced topics including security, monitoring, storage, autoscaling, CI/CD, troubleshooting, and best practices.

