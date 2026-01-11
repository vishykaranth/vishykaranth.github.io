# AWS ECS (Elastic Container Service) - In-Depth Guide

## Part 1: Fundamentals, Architecture, and Core Concepts

---

## Table of Contents

1. [Introduction to ECS](#1-introduction-to-ecs)
2. [ECS Architecture and Components](#2-ecs-architecture-and-components)
3. [Launch Types: Fargate vs EC2](#3-launch-types-fargate-vs-ec2)
4. [Task Definitions](#4-task-definitions)
5. [Tasks and Containers](#5-tasks-and-containers)
6. [Services](#6-services)
7. [Clusters](#7-clusters)
8. [Networking](#8-networking)

---

## 1. Introduction to ECS

### 1.1 What is ECS?

**AWS Elastic Container Service (ECS)** is a fully managed container orchestration service that makes it easy to run, stop, and manage Docker containers on a cluster of EC2 instances or using Fargate (serverless compute).

### 1.2 Key Characteristics

- **Fully Managed**: AWS handles the control plane, scheduling, and cluster management
- **Docker Compatible**: Runs standard Docker containers
- **Scalable**: Automatically scales from 1 to thousands of containers
- **Integrated**: Native integration with AWS services (ALB, CloudWatch, IAM, VPC)
- **Flexible**: Supports both serverless (Fargate) and EC2-based deployments
- **Cost-Effective**: Pay only for the resources you use

### 1.3 Use Cases

- **Microservices**: Deploy and manage microservices architectures
- **Batch Processing**: Run batch jobs and scheduled tasks
- **Web Applications**: Host web applications with auto-scaling
- **CI/CD Pipelines**: Run build and test containers
- **Hybrid Workloads**: Mix Fargate and EC2 launch types in the same cluster

### 1.4 ECS vs Other Container Services

| Feature | ECS | EKS | Docker Swarm | Kubernetes (Self-Managed) |
|---------|-----|-----|--------------|-------------------------|
| **Management** | Fully managed | Managed control plane | Self-managed | Self-managed |
| **Complexity** | Low | Medium | Low | High |
| **Kubernetes** | No | Yes | No | Yes |
| **AWS Integration** | Native | Good | Limited | Limited |
| **Best For** | AWS-native workloads | Kubernetes workloads | Simple orchestration | Full control |

---

## 2. ECS Architecture and Components

### 2.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS Account                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              ECS Cluster                              │  │
│  │                                                        │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │         ECS Control Plane (Managed by AWS)     │  │  │
│  │  │  - Task Scheduling                              │  │  │
│  │  │  - Service Management                           │  │  │
│  │  │  - Cluster State                                │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  │                                                        │  │
│  │  ┌────────────────────────────────────────────────┐  │  │
│  │  │         Compute Layer                           │  │  │
│  │  │                                                  │  │  │
│  │  │  ┌──────────┐  ┌──────────┐  ┌──────────┐     │  │  │
│  │  │  │  Fargate │  │  EC2     │  │  EC2     │     │  │  │
│  │  │  │  Tasks   │  │ Instance │  │ Instance │     │  │  │
│  │  │  │          │  │          │  │          │     │  │  │
│  │  │  │ ┌──────┐ │  │ ┌──────┐ │  │ ┌──────┐ │     │  │  │
│  │  │  │ │Task 1│ │  │ │Task 2│ │  │ │Task 3│ │     │  │  │
│  │  │  │ └──────┘ │  │ └──────┘ │  │ └──────┘ │     │  │  │
│  │  │  └──────────┘  └──────────┘  └──────────┘     │  │  │
│  │  └────────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         Supporting Services                           │  │
│  │  - ECR (Container Registry)                          │  │
│  │  - ALB/NLB (Load Balancing)                         │  │
│  │  - CloudWatch (Monitoring)                           │  │
│  │  - IAM (Security)                                    │  │
│  │  - VPC (Networking)                                  │  │
│  └──────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 2.2 Core Components

#### Cluster
A logical grouping of tasks or services. Think of it as a "namespace" for your containers.

**Characteristics**:
- Can span multiple Availability Zones
- Can contain both Fargate and EC2 tasks
- Has its own VPC and networking configuration
- Managed by AWS control plane

#### Task Definition
A blueprint that describes how containers should run. Similar to a Docker Compose file.

**Contains**:
- Container images
- CPU and memory requirements
- Environment variables
- Port mappings
- Volume mounts
- IAM roles
- Logging configuration

#### Task
A running instance of a Task Definition. One task can contain multiple containers (sidecar pattern).

**Characteristics**:
- Has its own network interface (in awsvpc mode)
- Gets a private IP address
- Can have multiple containers
- Lifecycle managed by ECS

#### Service
Manages long-running tasks, ensuring desired count is maintained.

**Responsibilities**:
- Maintains desired task count
- Replaces unhealthy tasks
- Handles deployments (rolling, blue/green, canary)
- Integrates with load balancers
- Auto-scaling based on metrics

#### Container Instance (EC2 Launch Type)
EC2 instance that's part of an ECS cluster and runs the ECS agent.

**Components**:
- ECS Agent: Communicates with ECS control plane
- Docker daemon: Runs containers
- Operating system: Amazon Linux, Ubuntu, Windows, etc.

---

## 3. Launch Types: Fargate vs EC2

### 3.1 Fargate (Serverless)

**What is Fargate?**
Fargate is a serverless compute engine for containers that eliminates the need to manage EC2 instances.

#### Key Features

**Pros**:
- ✅ **No Server Management**: AWS manages all infrastructure
- ✅ **Automatic Scaling**: Infrastructure scales automatically
- ✅ **Pay Per Task**: Pay only for vCPU and memory used
- ✅ **Enhanced Security**: Task-level isolation
- ✅ **Quick Start**: No capacity planning needed
- ✅ **Simplified Operations**: No OS patching or Docker daemon management

**Cons**:
- ❌ **Higher Cost**: More expensive per compute unit than EC2
- ❌ **No GPU Support**: Cannot use GPU instances
- ❌ **Limited CPU/Memory Combinations**: Fixed combinations available
- ❌ **No Persistent Storage**: Only EFS supported (no EBS)
- ❌ **No SSH Access**: Cannot SSH into containers
- ❌ **Limited Customization**: Cannot customize instance types

#### Fargate CPU and Memory Combinations

| CPU (vCPU) | Memory Range (GB) | Example Combinations |
|------------|-------------------|---------------------|
| 0.25 | 0.5 - 2 | 0.25 vCPU / 0.5 GB, 0.25 vCPU / 1 GB, 0.25 vCPU / 2 GB |
| 0.5 | 1 - 4 | 0.5 vCPU / 1 GB, 0.5 vCPU / 2 GB, 0.5 vCPU / 4 GB |
| 1 | 2 - 8 | 1 vCPU / 2 GB, 1 vCPU / 4 GB, 1 vCPU / 8 GB |
| 2 | 4 - 16 | 2 vCPU / 4 GB, 2 vCPU / 8 GB, 2 vCPU / 16 GB |
| 4 | 8 - 30 | 4 vCPU / 8 GB, 4 vCPU / 16 GB, 4 vCPU / 30 GB |
| 8 | 16 - 60 | 8 vCPU / 16 GB, 8 vCPU / 32 GB, 8 vCPU / 60 GB |
| 16 | 32 - 120 | 16 vCPU / 32 GB, 16 vCPU / 64 GB, 16 vCPU / 120 GB |

**Note**: Memory must be within the valid range for the selected CPU.

#### Fargate Pricing Model

```
Cost = (vCPU hours × vCPU price) + (Memory GB hours × Memory price)

Example:
- Task: 1 vCPU, 2 GB memory
- Runtime: 1 hour
- Cost: (1 × $0.04048) + (2 × $0.004445) = $0.04937/hour
```

### 3.2 EC2 Launch Type

**What is EC2 Launch Type?**
Running ECS tasks on EC2 instances that you manage.

#### Key Features

**Pros**:
- ✅ **Cost Effective**: Lower cost for steady workloads
- ✅ **GPU Support**: Can use GPU instances
- ✅ **Full Control**: Choose instance types, AMIs, configurations
- ✅ **Persistent Storage**: EBS, EFS, and instance store support
- ✅ **Spot Instances**: Use Spot instances for cost savings
- ✅ **SSH Access**: Can SSH into instances for debugging
- ✅ **Custom AMIs**: Use custom Amazon Machine Images

**Cons**:
- ❌ **Server Management**: Must manage EC2 instances
- ❌ **Capacity Planning**: Need to plan and provision capacity
- ❌ **OS Patching**: Responsible for OS updates
- ❌ **Docker Management**: Manage Docker daemon
- ❌ **Agent Updates**: Manage ECS agent updates
- ❌ **More Complex**: More operational overhead

#### EC2 Instance Types for ECS

**General Purpose**:
- `t3`, `t3a`: Burstable performance, good for variable workloads
- `m5`, `m5a`: Balanced compute, memory, and network

**Compute Optimized**:
- `c5`, `c5a`: High-performance processors, good for CPU-intensive tasks

**Memory Optimized**:
- `r5`, `r5a`: High memory-to-vCPU ratio, good for memory-intensive tasks

**Storage Optimized**:
- `i3`: High IOPS SSD storage

**GPU Instances**:
- `p3`, `p4`: GPU instances for machine learning and graphics

### 3.3 Comparison Table

| Feature | Fargate | EC2 |
|---------|---------|-----|
| **Infrastructure Management** | None (AWS managed) | Full (you manage) |
| **Pricing Model** | Per task (vCPU + memory per second) | Per EC2 instance (hourly) |
| **Startup Time** | ~30-60 seconds | Depends on instance (minutes) |
| **GPU Support** | ❌ No | ✅ Yes |
| **Persistent Storage** | EFS only | EBS, EFS, Instance Store |
| **Max Task Size** | 16 vCPU, 120 GB | Instance limit |
| **Cost for Steady Workloads** | Higher | Lower |
| **Cost for Variable Workloads** | Lower (pay per use) | Higher (pay for idle) |
| **SSH Access** | ❌ No | ✅ Yes |
| **Custom AMIs** | ❌ No | ✅ Yes |
| **Spot Instances** | ✅ Fargate Spot | ✅ EC2 Spot |
| **Best For** | Variable workloads, small teams | Steady workloads, cost optimization |

### 3.4 When to Use Each

#### Use Fargate When:
- ✅ Workloads are variable or unpredictable
- ✅ Team is small or has limited DevOps expertise
- ✅ Quick scaling is needed
- ✅ Security isolation is critical
- ✅ You want to focus on application code, not infrastructure
- ✅ Workloads don't need GPUs or persistent local storage

#### Use EC2 When:
- ✅ Cost optimization is priority (steady workloads)
- ✅ You need GPUs for ML/graphics workloads
- ✅ You need large tasks (>16 vCPU or >120 GB memory)
- ✅ You need specific instance types or configurations
- ✅ You want to use Spot instances for big savings
- ✅ You need local persistent storage (EBS)
- ✅ You need SSH access for debugging

---

## 4. Task Definitions

### 4.1 What is a Task Definition?

A Task Definition is a JSON document that describes one or more containers that form your application. It's like a blueprint for how containers should run.

### 4.2 Task Definition Structure

```json
{
  "family": "my-app",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::123456789:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789:role/ecsTaskRole",
  "containerDefinitions": [
    {
      "name": "web",
      "image": "123456789.dkr.ecr.us-east-1.amazonaws.com/my-app:latest",
      "essential": true,
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "environment": [
        {"name": "NODE_ENV", "value": "production"}
      ],
      "secrets": [
        {
          "name": "DB_PASSWORD",
          "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:my-db-secret:password::"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/my-app",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "healthCheck": {
        "command": ["CMD-SHELL", "curl -f http://localhost:80/health || exit 1"],
        "interval": 30,
        "timeout": 5,
        "retries": 3,
        "startPeriod": 60
      }
    }
  ]
}
```

### 4.3 Key Task Definition Parameters

#### Family
Unique name for your task definition. Used to group versions.

```json
"family": "my-web-app"
```

#### Network Mode
Determines how containers are networked.

**Options**:
- **awsvpc**: Each task gets its own ENI (required for Fargate, recommended for EC2)
- **bridge**: Docker's built-in virtual network (EC2 only)
- **host**: Uses EC2 host's network (EC2 only, high performance)
- **none**: No network connectivity (batch jobs)

```json
"networkMode": "awsvpc"
```

#### CPU and Memory (Fargate)

Must use valid combinations. Specified as strings.

```json
"cpu": "256",      // 0.25 vCPU
"memory": "512"    // 512 MB
```

#### Execution Role
IAM role used by ECS to pull images, push logs, retrieve secrets.

```json
"executionRoleArn": "arn:aws:iam::123456789:role/ecsTaskExecutionRole"
```

**Permissions Needed**:
- `ecr:GetAuthorizationToken`
- `ecr:BatchCheckLayerAvailability`
- `ecr:GetDownloadUrlForLayer`
- `ecr:BatchGetImage`
- `logs:CreateLogStream`
- `logs:PutLogEvents`
- `secretsmanager:GetSecretValue`
- `ssm:GetParameters`

#### Task Role
IAM role assumed by your application containers to access AWS services.

```json
"taskRoleArn": "arn:aws:iam::123456789:role/ecsTaskRole"
```

**Use Cases**:
- Access S3 buckets
- Query DynamoDB
- Call other AWS services
- Send messages to SQS

### 4.4 Container Definitions

#### Basic Container Definition

```json
{
  "name": "web",
  "image": "nginx:latest",
  "essential": true,
  "memory": 512,
  "memoryReservation": 256,
  "cpu": 256
}
```

#### Essential vs Non-Essential

**Essential Container** (`essential: true`):
- If it stops, the entire task stops
- At least one container must be essential
- Use for main application containers

**Non-Essential Container** (`essential: false`):
- If it stops, task continues running
- Use for sidecars, logging agents, monitoring
- Optional helper containers

#### Port Mappings

```json
"portMappings": [
  {
    "containerPort": 80,
    "hostPort": 80,        // Only for bridge/host mode
    "protocol": "tcp"
  }
]
```

**For awsvpc mode**: Only `containerPort` is needed (no `hostPort`).

#### Environment Variables

**Hardcoded** (non-sensitive):
```json
"environment": [
  {"name": "NODE_ENV", "value": "production"},
  {"name": "API_URL", "value": "https://api.example.com"}
]
```

**From Secrets Manager**:
```json
"secrets": [
  {
    "name": "DB_PASSWORD",
    "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:my-db-secret:password::"
  }
]
```

**From Parameter Store**:
```json
"secrets": [
  {
    "name": "API_KEY",
    "valueFrom": "arn:aws:ssm:us-east-1:123456789:parameter/my-app/api-key"
  }
]
```

#### Logging Configuration

**CloudWatch Logs**:
```json
"logConfiguration": {
  "logDriver": "awslogs",
  "options": {
    "awslogs-group": "/ecs/my-app",
    "awslogs-region": "us-east-1",
    "awslogs-stream-prefix": "ecs",
    "awslogs-create-group": "true"
  }
}
```

**Other Log Drivers**:
- `splunk`: Send to Splunk
- `fluentd`: Send to Fluentd
- `json-file`: Local JSON files
- `awsfirelens`: Advanced log routing

#### Health Checks

```json
"healthCheck": {
  "command": ["CMD-SHELL", "curl -f http://localhost:80/health || exit 1"],
  "interval": 30,        // Seconds between checks
  "timeout": 5,          // Seconds to wait for response
  "retries": 3,          // Consecutive failures before unhealthy
  "startPeriod": 60     // Grace period before counting failures
}
```

**Health Check Flow**:
1. Container starts
2. `startPeriod` grace period (60s) - failures don't count
3. Health checks run every `interval` (30s)
4. If check fails, wait `timeout` (5s) then retry
5. After `retries` (3) consecutive failures → container marked unhealthy
6. ECS stops unhealthy container and starts new one

### 4.5 Multi-Container Task Definitions

#### Sidecar Pattern

```json
"containerDefinitions": [
  {
    "name": "app",
    "image": "my-app:latest",
    "essential": true,
    "portMappings": [{"containerPort": 3000}]
  },
  {
    "name": "log-agent",
    "image": "fluentd:latest",
    "essential": false,
    "dependsOn": [
      {"containerName": "app", "condition": "START"}
    ]
  }
]
```

#### Ambassador Pattern

```json
"containerDefinitions": [
  {
    "name": "app",
    "image": "my-app:latest",
    "essential": true
  },
  {
    "name": "proxy",
    "image": "envoy:latest",
    "essential": true,
    "portMappings": [{"containerPort": 8080}],
    "dependsOn": [
      {"containerName": "app", "condition": "HEALTHY"}
    ]
  }
]
```

#### Init Container Pattern

```json
"containerDefinitions": [
  {
    "name": "init",
    "image": "busybox:latest",
    "essential": false,
    "command": ["sh", "-c", "echo 'Initializing...' && sleep 5"]
  },
  {
    "name": "app",
    "image": "my-app:latest",
    "essential": true,
    "dependsOn": [
      {"containerName": "init", "condition": "SUCCESS"}
    ]
  }
]
```

### 4.6 Volumes and Storage

#### EFS (Elastic File System)

```json
"volumes": [
  {
    "name": "efs-volume",
    "efsVolumeConfiguration": {
      "fileSystemId": "fs-12345678",
      "rootDirectory": "/",
      "transitEncryption": "ENABLED",
      "authorizationConfig": {
        "accessPointId": "fsap-12345678",
        "iam": "ENABLED"
      }
    }
  }
],
"containerDefinitions": [
  {
    "name": "app",
    "mountPoints": [
      {
        "sourceVolume": "efs-volume",
        "containerPath": "/data",
        "readOnly": false
      }
    ]
  }
]
```

#### Bind Mounts (EC2 only)

```json
"volumes": [
  {"name": "shared-data"}
],
"containerDefinitions": [
  {
    "name": "writer",
    "mountPoints": [
      {"sourceVolume": "shared-data", "containerPath": "/data"}
    ]
  },
  {
    "name": "reader",
    "mountPoints": [
      {"sourceVolume": "shared-data", "containerPath": "/data", "readOnly": true}
    ]
  }
]
```

---

## 5. Tasks and Containers

### 5.1 What is a Task?

A Task is a running instance of a Task Definition. It represents one or more containers running together.

### 5.2 Task Lifecycle

```
┌──────────┐
│ PENDING  │ ← Task definition created, waiting for resources
└────┬─────┘
     │
     ▼
┌──────────┐
│ ACTIVATING│ ← Resources allocated, containers starting
└────┬─────┘
     │
     ▼
┌──────────┐
│ RUNNING  │ ← Containers running
└────┬─────┘
     │
     ├─→ ┌──────────┐
     │   │ STOPPING │ ← Stop command received
     │   └────┬─────┘
     │        │
     │        ▼
     │   ┌──────────┐
     │   │ STOPPED  │ ← Task stopped
     │   └──────────┘
     │
     └─→ ┌──────────┐
         │ DEPROVISIONING│ ← Resources being released (Fargate)
         └────┬─────┘
              │
              ▼
         ┌──────────┐
         │ STOPPED  │
         └──────────┘
```

### 5.3 Task States

**PENDING**:
- Task definition registered
- Waiting for resources (CPU, memory, ENI)
- No containers started yet

**ACTIVATING**:
- Resources allocated
- Containers being pulled and started
- Health checks in grace period

**RUNNING**:
- All containers running
- Health checks passing
- Task receiving traffic (if part of service)

**STOPPING**:
- Stop command received
- Containers being gracefully stopped
- SIGTERM sent to containers

**STOPPED**:
- All containers stopped
- Resources released
- Task no longer running

### 5.4 Container States

**CREATED**: Container created but not started
**RUNNING**: Container is running
**STOPPED**: Container stopped
**PENDING**: Container waiting to start

### 5.5 Task Placement

#### Placement Strategies (EC2)

**spread**: Distribute tasks evenly
```json
"placementStrategy": [
  {"type": "spread", "field": "attribute:ecs.availability-zone"}
]
```

**binpack**: Pack tasks tightly to minimize instances
```json
"placementStrategy": [
  {"type": "binpack", "field": "memory"}
]
```

**random**: Place randomly
```json
"placementStrategy": [
  {"type": "random"}
]
```

#### Placement Constraints (EC2)

**distinctInstance**: One task per EC2 instance
```json
"placementConstraints": [
  {"type": "distinctInstance"}
]
```

**memberOf**: Place on instances matching expression
```json
"placementConstraints": [
  {
    "type": "memberOf",
    "expression": "attribute:ecs.instance-type =~ t3.*"
  }
]
```

---

## 6. Services

### 6.1 What is a Service?

A Service maintains a specified number of running tasks (desired count) and automatically replaces any that fail or stop.

### 6.2 Service Types

#### Replica Service (Most Common)

Maintains a desired count of identical tasks.

```json
{
  "serviceName": "web-service",
  "taskDefinition": "my-app:10",
  "desiredCount": 3,
  "launchType": "FARGATE",
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["subnet-123", "subnet-456"],
      "securityGroups": ["sg-123"],
      "assignPublicIp": "ENABLED"
    }
  },
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:...",
      "containerName": "web",
      "containerPort": 80
    }
  ]
}
```

**Characteristics**:
- Maintains desired count across AZs
- Replaces failed tasks automatically
- Integrates with load balancers
- Supports auto-scaling

#### Daemon Service (EC2 only)

Runs exactly one task per EC2 instance in the cluster.

```json
{
  "serviceName": "monitoring-agent",
  "taskDefinition": "monitoring:5",
  "schedulingStrategy": "DAEMON",
  "launchType": "EC2"
}
```

**Use Cases**:
- Log collection agents
- Monitoring agents
- Security scanners
- Anti-virus software

**Characteristics**:
- Automatically added to new instances
- One task per instance
- Not available on Fargate

### 6.3 Service Scheduler

The Service Scheduler is responsible for:
- Maintaining desired task count
- Replacing unhealthy tasks
- Handling deployments
- Registering with load balancers
- Respecting placement constraints
- Scaling based on auto-scaling rules

### 6.4 Service Deployment Configuration

```json
{
  "deploymentConfiguration": {
    "minimumHealthyPercent": 50,
    "maximumPercent": 200,
    "deploymentCircuitBreaker": {
      "enable": true,
      "rollback": true
    }
  }
}
```

**minimumHealthyPercent**: 50
- During deployment, keep at least 50% of tasks running
- Example: 10 tasks → at least 5 running during deploy

**maximumPercent**: 200
- Can temporarily run up to 200% of desired count
- Example: 10 tasks → can run up to 20 during deploy

**Deployment Flow**:
```
Before: [Task v1] [Task v1] [Task v1] [Task v1] [Task v1] (5 tasks)
Step 1: [Task v1] [Task v1] [Task v1] [Task v2] [Task v2] (5 tasks, 2 new)
Step 2: [Task v2] [Task v2] [Task v2] [Task v2] [Task v2] (5 tasks, all new)
```

**Circuit Breaker**:
- Monitors deployment health
- If >50% of new tasks fail → automatically rollback
- Prevents bad deployments from taking down service

---

## 7. Clusters

### 7.1 What is a Cluster?

A Cluster is a logical grouping of tasks or services. It provides isolation and resource management.

### 7.2 Cluster Types

#### Fargate Cluster
- No EC2 instances to manage
- Tasks run on Fargate
- Simplest cluster type

#### EC2 Cluster
- Contains EC2 instances
- Tasks run on EC2
- More control, more management

#### Mixed Cluster
- Contains both Fargate and EC2 capacity
- Can run both launch types
- Flexible resource allocation

### 7.3 Cluster Configuration

```json
{
  "clusterName": "production-cluster",
  "capacityProviders": ["FARGATE", "FARGATE_SPOT"],
  "defaultCapacityProviderStrategy": [
    {
      "capacityProvider": "FARGATE_SPOT",
      "weight": 70,
      "base": 0
    },
    {
      "capacityProvider": "FARGATE",
      "weight": 30,
      "base": 2
    }
  ],
  "clusterSettings": [
    {
      "name": "containerInsights",
      "value": "enabled"
    }
  ]
}
```

### 7.4 Capacity Providers

Capacity Providers determine how tasks are placed and how the cluster scales.

#### Fargate Capacity Provider
- Serverless capacity
- Automatic scaling
- Pay per task

#### Fargate Spot Capacity Provider
- Up to 70% discount
- Can be interrupted
- Good for fault-tolerant workloads

#### EC2 Capacity Provider
- Links to Auto Scaling Group
- Automatic instance scaling
- Supports Spot instances

---

## 8. Networking

### 8.1 Network Modes

#### awsvpc (Recommended)

Each task gets its own Elastic Network Interface (ENI).

**Characteristics**:
- Each task has unique private IP
- Security groups at task level
- VPC Flow Logs per task
- Direct VPC connectivity
- Required for Fargate

**Architecture**:
```
Task
┌─────────────────┐
│ Container (Port 80)│
└────────┬─────────┘
         │
    ┌────▼────┐
    │   ENI   │
    │ 10.0.1.5│
    └─────────┘
```

#### bridge (EC2 only)

Docker's built-in virtual network.

**Characteristics**:
- Tasks share host network
- Dynamic port mapping
- Less isolation
- Legacy support

#### host (EC2 only)

Uses EC2 host's network directly.

**Characteristics**:
- Highest performance
- No network translation
- Less isolation
- Port conflicts possible

#### none

No network connectivity.

**Use Cases**:
- Batch jobs with no network needs
- Isolated processing tasks

### 8.2 VPC Configuration

#### Subnet Configuration

```json
{
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": [
        "subnet-12345678",  // AZ-1
        "subnet-87654321"   // AZ-2
      ],
      "securityGroups": ["sg-12345678"],
      "assignPublicIp": "DISABLED"  // or "ENABLED"
    }
  }
}
```

**Best Practices**:
- Use private subnets for tasks
- Use public subnets only if needed
- Spread across multiple AZs
- Use NAT Gateway for outbound internet

#### Security Groups

**ALB Security Group**:
```
Inbound:
  - Port 80 from 0.0.0.0/0
  - Port 443 from 0.0.0.0/0
Outbound:
  - All traffic to ECS task security group
```

**ECS Task Security Group**:
```
Inbound:
  - Port 80 from ALB security group
  - Port 443 from ALB security group
Outbound:
  - Port 443 to 0.0.0.0/0 (HTTPS)
  - Port 443 to VPC endpoints
```

### 8.3 VPC Endpoints (Private Access)

Instead of NAT Gateway, use VPC Endpoints for private access to AWS services.

**Required Endpoints for Fargate**:
- `com.amazonaws.region.ecr.api` - ECR API
- `com.amazonaws.region.ecr.dkr` - ECR Docker
- `com.amazonaws.region.s3` - S3 (for ECR layers)
- `com.amazonaws.region.logs` - CloudWatch Logs

**Benefits**:
- No internet gateway needed
- No NAT Gateway costs
- Better security (private connectivity)
- Lower latency

### 8.4 Service Discovery

AWS Cloud Map integration for service-to-service communication.

```json
{
  "serviceRegistries": [
    {
      "registryArn": "arn:aws:servicediscovery:us-east-1:123456789:service/srv-xxx",
      "containerName": "web",
      "containerPort": 80
    }
  ]
}
```

**DNS-Based Discovery**:
```
Service A queries: service-b.local
DNS returns: 10.0.1.5
Service A connects to Service B
```

---

## Summary of Part 1

**Key Takeaways**:
1. **ECS** is a fully managed container orchestration service
2. **Fargate** is serverless (no server management), **EC2** gives more control
3. **Task Definitions** are blueprints describing how containers run
4. **Tasks** are running instances of task definitions
5. **Services** maintain desired count and handle deployments
6. **Clusters** provide logical grouping and resource management
7. **Networking** uses awsvpc mode for task-level isolation

**Next**: Part 2 will cover load balancing, auto-scaling, deployment strategies, monitoring, security, and best practices.

