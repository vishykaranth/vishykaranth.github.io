# AWS ECS (Elastic Container Service) - In-Depth Guide

## Part 2: Advanced Topics, Operations, and Best Practices

---

## Table of Contents

1. [Load Balancing](#1-load-balancing)
2. [Auto Scaling](#2-auto-scaling)
3. [Deployment Strategies](#3-deployment-strategies)
4. [Monitoring and Logging](#4-monitoring-and-logging)
5. [Security](#5-security)
6. [Service Discovery](#6-service-discovery)
7. [Cost Optimization](#7-cost-optimization)
8. [Troubleshooting](#8-troubleshooting)
9. [Best Practices](#9-best-practices)
10. [Real-World Examples](#10-real-world-examples)

---

## 1. Load Balancing

### 1.1 Load Balancer Types

#### Application Load Balancer (ALB)

**Use Cases**:
- Web applications (HTTP/HTTPS)
- REST APIs
- Microservices
- Content-based routing

**Features**:
- Layer 7 (HTTP/HTTPS) load balancing
- Path-based routing
- Host-based routing
- SSL/TLS termination
- WebSocket support
- HTTP/2 support

**Configuration**:
```json
{
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:...:targetgroup/web-tg/...",
      "containerName": "web",
      "containerPort": 80
    }
  ]
}
```

#### Network Load Balancer (NLB)

**Use Cases**:
- High-performance applications
- TCP/UDP traffic
- Static IP addresses
- Extreme low latency

**Features**:
- Layer 4 (TCP/UDP) load balancing
- Static IP addresses
- Ultra-low latency
- High throughput
- Connection-based routing

#### Gateway Load Balancer (GWLB)

**Use Cases**:
- Network appliances (firewalls, intrusion detection)
- Third-party security appliances
- Transparent network inspection

### 1.2 Target Group Types

#### IP Target Type (Fargate/awsvpc)

```json
{
  "targetType": "ip",
  "port": 80,
  "protocol": "HTTP",
  "healthCheckPath": "/health"
}
```

**Characteristics**:
- Registers tasks by IP address
- Required for Fargate
- Required for awsvpc network mode
- Each task gets its own IP

#### Instance Target Type (EC2 bridge/host)

```json
{
  "targetType": "instance",
  "port": 32768,
  "protocol": "HTTP"
}
```

**Characteristics**:
- Registers by EC2 instance ID
- Used with bridge/host network mode
- Dynamic port mapping
- Multiple tasks per instance

### 1.3 Health Checks

#### ALB Health Checks

```json
{
  "healthCheckProtocol": "HTTP",
  "healthCheckPath": "/health",
  "healthCheckIntervalSeconds": 30,
  "healthCheckTimeoutSeconds": 5,
  "healthyThresholdCount": 2,
  "unhealthyThresholdCount": 3
}
```

**Health Check Flow**:
```
1. ALB sends health check request every 30s
2. Waits 5s for response
3. If healthy response → increment healthy count
4. If unhealthy response → increment unhealthy count
5. After 2 consecutive healthy → mark as healthy
6. After 3 consecutive unhealthy → mark as unhealthy
7. Unhealthy targets removed from rotation
```

#### Container Health Checks

```json
{
  "healthCheck": {
    "command": ["CMD-SHELL", "curl -f http://localhost:80/health || exit 1"],
    "interval": 30,
    "timeout": 5,
    "retries": 3,
    "startPeriod": 60
  }
}
```

**Two-Level Health Checks**:
- **Container Health Check**: ECS monitors container health, restarts if unhealthy
- **ALB Health Check**: ALB monitors target health, removes from rotation if unhealthy

### 1.4 Load Balancer Integration

#### Service with ALB

```json
{
  "serviceName": "web-service",
  "taskDefinition": "my-app:10",
  "desiredCount": 3,
  "loadBalancers": [
    {
      "targetGroupArn": "arn:aws:elasticloadbalancing:...:targetgroup/web-tg/...",
      "containerName": "web",
      "containerPort": 80
    }
  ],
  "healthCheckGracePeriodSeconds": 60
}
```

**Integration Flow**:
```
1. Service starts tasks
2. Tasks register with target group
3. ALB health checks begin after grace period
4. Healthy tasks receive traffic
5. Unhealthy tasks removed from rotation
6. Service replaces unhealthy tasks
```

---

## 2. Auto Scaling

### 2.1 Types of Auto Scaling

#### Service Auto Scaling

Scales the number of tasks in a service.

**Metrics**:
- `ECSServiceAverageCPUUtilization`
- `ECSServiceAverageMemoryUtilization`
- `ALBRequestCountPerTarget`
- Custom CloudWatch metrics

#### Cluster Auto Scaling (EC2)

Scales the number of EC2 instances in the cluster.

**Components**:
- Auto Scaling Group
- Capacity Provider
- ECS Cluster Auto Scaling

### 2.2 Service Auto Scaling Configuration

#### Target Tracking Scaling

```json
{
  "scalableTarget": {
    "ServiceNamespace": "ecs",
    "ResourceId": "service/my-cluster/web-service",
    "ScalableDimension": "ecs:service:DesiredCount",
    "MinCapacity": 2,
    "MaxCapacity": 10
  },
  "scalingPolicies": [
    {
      "PolicyName": "cpu-target-tracking",
      "PolicyType": "TargetTrackingScaling",
      "TargetTrackingScalingPolicyConfiguration": {
        "TargetValue": 70.0,
        "PredefinedMetricSpecification": {
          "PredefinedMetricType": "ECSServiceAverageCPUUtilization"
        },
        "ScaleInCooldown": 120,
        "ScaleOutCooldown": 60
      }
    }
  ]
}
```

**Behavior**:
- Keeps CPU at 70%
- If CPU > 70% → Add tasks
- If CPU < 70% → Remove tasks
- Cooldown periods prevent rapid scaling

#### Step Scaling

```json
{
  "PolicyName": "step-scaling",
  "PolicyType": "StepScaling",
  "StepScalingPolicyConfiguration": {
    "AdjustmentType": "ChangeInCapacity",
    "StepAdjustments": [
      {
        "MetricIntervalLowerBound": 0,
        "MetricIntervalUpperBound": 10,
        "ScalingAdjustment": 1
      },
      {
        "MetricIntervalLowerBound": 10,
        "MetricIntervalUpperBound": 20,
        "ScalingAdjustment": 2
      },
      {
        "MetricIntervalLowerBound": 20,
        "ScalingAdjustment": 3
      }
    ],
    "Cooldown": 300
  }
}
```

**Behavior**:
- CPU 70-80% → Add 1 task
- CPU 80-90% → Add 2 tasks
- CPU > 90% → Add 3 tasks

#### Scheduled Scaling

```json
{
  "PolicyName": "scheduled-scaling",
  "PolicyType": "TargetTrackingScaling",
  "ScheduledActions": [
    {
      "ScheduledActionName": "scale-up-morning",
      "Schedule": "cron(0 9 * * ? *)",
      "ScalableTargetAction": {
        "MinCapacity": 5,
        "MaxCapacity": 20
      }
    },
    {
      "ScheduledActionName": "scale-down-evening",
      "Schedule": "cron(0 18 * * ? *)",
      "ScalableTargetAction": {
        "MinCapacity": 2,
        "MaxCapacity": 10
      }
    }
  ]
}
```

**Use Cases**:
- Scale up during business hours
- Scale down during off-hours
- Handle known traffic patterns

### 2.3 Capacity Providers (EC2 Auto Scaling)

#### EC2 Capacity Provider

```json
{
  "capacityProviders": ["my-ec2-capacity-provider"],
  "defaultCapacityProviderStrategy": [
    {
      "capacityProvider": "my-ec2-capacity-provider",
      "weight": 1,
      "base": 0
    }
  ]
}
```

**Configuration**:
- Links to Auto Scaling Group
- Automatically scales EC2 instances
- Ensures capacity for tasks
- Supports Spot instances

#### Fargate Spot Capacity Provider

```json
{
  "capacityProviders": ["FARGATE_SPOT"],
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
  ]
}
```

**Strategy**:
- 70% of tasks on Fargate Spot (cheaper, can be interrupted)
- 30% of tasks on Fargate (on-demand, reliable)
- Base of 2 ensures at least 2 tasks on Fargate

### 2.4 Auto Scaling Best Practices

**Metrics Selection**:
- Use CPU for CPU-bound applications
- Use Memory for memory-bound applications
- Use RequestCount for request-based scaling
- Use custom metrics for business logic

**Cooldown Periods**:
- Scale-out cooldown: 60-120 seconds (allow time for tasks to start)
- Scale-in cooldown: 300-600 seconds (prevent rapid scale-in)

**Min/Max Capacity**:
- Set minimum based on availability requirements
- Set maximum based on cost constraints
- Consider burst capacity for unexpected traffic

---

## 3. Deployment Strategies

### 3.1 Rolling Update (Default)

**How it Works**:
- Gradually replaces old tasks with new tasks
- Maintains service availability
- Configurable minimum/maximum healthy percent

**Configuration**:
```json
{
  "deploymentConfiguration": {
    "minimumHealthyPercent": 50,
    "maximumPercent": 200
  }
}
```

**Deployment Flow**:
```
Before: [v1] [v1] [v1] [v1] [v1] (5 tasks, 100%)
Step 1: [v1] [v1] [v1] [v2] [v2] (5 tasks, 60% v1, 40% v2)
Step 2: [v2] [v2] [v2] [v2] [v2] (5 tasks, 100% v2)
```

**Pros**:
- Zero downtime
- Gradual rollout
- Automatic rollback on failure

**Cons**:
- Both versions run simultaneously
- Potential compatibility issues

### 3.2 Blue/Green Deployment

**How it Works**:
- Deploys new version to separate target group
- Tests new version
- Switches traffic to new version
- Keeps old version for rollback

**Architecture**:
```
ALB Listener
    │
    ├─→ Blue Target Group (v1) ← Current traffic
    │   └─→ [v1] [v1] [v1]
    │
    └─→ Green Target Group (v2) ← New deployment
        └─→ [v2] [v2] [v2]
```

**Implementation**:
- Use CodeDeploy for ECS blue/green deployments
- Or manually manage two target groups
- Switch traffic using ALB listener rules

**Pros**:
- Instant rollback
- Complete isolation
- Easy testing

**Cons**:
- Requires double capacity
- More complex setup

### 3.3 Canary Deployment

**How it Works**:
- Deploys new version to small percentage of traffic
- Monitors metrics
- Gradually increases traffic
- Rolls back if issues detected

**Traffic Split**:
```
90% → v1 (stable)
10% → v2 (canary)
    │
    ├─→ If healthy: Increase to 25%, 50%, 100%
    └─→ If unhealthy: Rollback to 100% v1
```

**Implementation**:
- Use ALB weighted target groups
- Use CloudWatch alarms for monitoring
- Use Step Functions for orchestration

**Pros**:
- Low risk
- Gradual validation
- Easy rollback

**Cons**:
- Complex setup
- Requires monitoring

### 3.4 Circuit Breaker Deployment

**How it Works**:
- Monitors deployment health
- Automatically rolls back if failure rate exceeds threshold
- Prevents bad deployments from taking down service

**Configuration**:
```json
{
  "deploymentConfiguration": {
    "deploymentCircuitBreaker": {
      "enable": true,
      "rollback": true
    }
  }
}
```

**Behavior**:
- Monitors task health during deployment
- If >50% of new tasks fail → automatic rollback
- Prevents cascading failures

---

## 4. Monitoring and Logging

### 4.1 CloudWatch Metrics

#### Service Metrics

**ECSServiceAverageCPUUtilization**:
- Average CPU utilization across all tasks
- Used for auto-scaling
- Range: 0-100%

**ECSServiceAverageMemoryUtilization**:
- Average memory utilization across all tasks
- Used for auto-scaling
- Range: 0-100%

**RunningTaskCount**:
- Number of running tasks
- Monitors service health

**PendingTaskCount**:
- Number of tasks waiting to start
- Indicates capacity issues

#### Cluster Metrics

**ActiveInstanceCount** (EC2):
- Number of active EC2 instances
- Monitors cluster capacity

**CPUReservation**:
- Percentage of CPU reserved
- Indicates resource utilization

**MemoryReservation**:
- Percentage of memory reserved
- Indicates resource utilization

### 4.2 CloudWatch Logs

#### Log Configuration

```json
{
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/my-app",
      "awslogs-region": "us-east-1",
      "awslogs-stream-prefix": "ecs",
      "awslogs-create-group": "true"
    }
  }
}
```

#### Log Groups and Streams

**Log Group**: `/ecs/my-app`
- Contains all logs for the application
- Retention policy configurable
- Encryption support

**Log Streams**: `ecs/web/1234567890-abcdef`
- One stream per container instance
- Automatic creation
- Real-time streaming

#### Log Retention

```bash
aws logs put-retention-policy \
  --log-group-name /ecs/my-app \
  --retention-in-days 30
```

**Options**:
- 1, 3, 5, 7, 14, 30, 60, 90, 120, 180, 365 days
- Never expire
- Cost consideration: Longer retention = higher cost

### 4.3 Container Insights

**What is Container Insights?**
Enhanced monitoring for ECS with automatic dashboards and metrics.

**Enable**:
```bash
aws ecs update-cluster-settings \
  --cluster my-cluster \
  --settings name=containerInsights,value=enabled
```

**Features**:
- Automatic dashboards
- Performance metrics
- Resource utilization graphs
- Anomaly detection
- Integration with CloudWatch Alarms

**Metrics Provided**:
- CPU utilization per container
- Memory utilization per container
- Network I/O
- Storage I/O
- Task and service metrics

### 4.4 CloudWatch Alarms

#### CPU Utilization Alarm

```json
{
  "AlarmName": "ecs-high-cpu",
  "MetricName": "CPUUtilization",
  "Namespace": "AWS/ECS",
  "Statistic": "Average",
  "Period": 300,
  "EvaluationPeriods": 2,
  "Threshold": 80,
  "ComparisonOperator": "GreaterThanThreshold",
  "AlarmActions": ["arn:aws:sns:us-east-1:123456789:alerts"]
}
```

#### Memory Utilization Alarm

```json
{
  "AlarmName": "ecs-high-memory",
  "MetricName": "MemoryUtilization",
  "Namespace": "AWS/ECS",
  "Statistic": "Average",
  "Period": 300,
  "EvaluationPeriods": 2,
  "Threshold": 85,
  "ComparisonOperator": "GreaterThanThreshold"
}
```

### 4.5 X-Ray Integration

**Distributed Tracing**:
- Track requests across services
- Identify performance bottlenecks
- Debug microservices

**Configuration**:
```json
{
  "containerDefinitions": [
    {
      "name": "app",
      "image": "my-app:latest",
      "environment": [
        {"name": "_X_AMZN_TRACE_ID", "value": ""}
      ]
    }
  ]
}
```

---

## 5. Security

### 5.1 IAM Roles

#### Task Execution Role

**Purpose**: Used by ECS to pull images, push logs, retrieve secrets.

**Permissions**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetAuthorizationToken",
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "logs:CreateLogStream",
        "logs:PutLogEvents",
        "secretsmanager:GetSecretValue"
      ],
      "Resource": "*"
    }
  ]
}
```

#### Task Role

**Purpose**: Used by your application containers to access AWS services.

**Example**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}
```

### 5.2 Secrets Management

#### Secrets Manager

```json
{
  "secrets": [
    {
      "name": "DB_PASSWORD",
      "valueFrom": "arn:aws:secretsmanager:us-east-1:123456789:secret:my-db-secret:password::"
    }
  ]
}
```

**Best Practices**:
- Use Secrets Manager for sensitive data
- Rotate secrets regularly
- Use least privilege for execution role
- Never hardcode secrets

#### Parameter Store

```json
{
  "secrets": [
    {
      "name": "API_KEY",
      "valueFrom": "arn:aws:ssm:us-east-1:123456789:parameter/my-app/api-key"
    }
  ]
}
```

**Use Cases**:
- Non-sensitive configuration
- Application settings
- Feature flags

### 5.3 Network Security

#### Security Groups

**Task Security Group**:
```
Inbound:
  - Port 80 from ALB security group
  - Port 443 from ALB security group
Outbound:
  - Port 443 to 0.0.0.0/0 (HTTPS)
  - Port 443 to VPC endpoints
```

**Best Practices**:
- Principle of least privilege
- Deny all by default
- Allow only necessary ports
- Use security group references

#### VPC Configuration

**Private Subnets**:
- Tasks in private subnets
- No direct internet access
- Use NAT Gateway or VPC Endpoints

**Public Subnets**:
- Only if needed
- Use with caution
- Additional security measures

### 5.4 Container Security

#### Read-Only Root Filesystem

```json
{
  "readonlyRootFilesystem": true
}
```

**Benefits**:
- Prevents file system modifications
- Reduces attack surface
- Compliance requirement

#### Non-Root User

```json
{
  "user": "1000:1000"
}
```

**Benefits**:
- Run containers as non-root
- Reduces privilege escalation risk
- Security best practice

#### Capabilities

```json
{
  "linuxParameters": {
    "capabilities": {
      "drop": ["ALL"],
      "add": ["NET_BIND_SERVICE"]
    }
  }
}
```

**Best Practices**:
- Drop all capabilities by default
- Add only necessary capabilities
- Minimize privileges

### 5.5 Image Security

#### ECR Image Scanning

**Automated Scanning**:
- Scans images for vulnerabilities
- Integrates with ECR
- Uses Common Vulnerabilities and Exposures (CVE) database

**Configuration**:
```bash
aws ecr put-image-scanning-configuration \
  --repository-name my-app \
  --image-scanning-configuration scanOnPush=true
```

#### Image Tagging

**Best Practices**:
- Use semantic versioning
- Tag with commit SHA
- Avoid `latest` tag in production
- Use immutable tags

---

## 6. Service Discovery

### 6.1 AWS Cloud Map

**What is Cloud Map?**
Service discovery service that automatically discovers services using DNS or API.

#### DNS-Based Service Discovery

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

**How it Works**:
```
Service A queries: web-service.local
DNS returns: 10.0.1.5, 10.0.2.6, 10.0.3.7
Service A connects to one of the IPs
```

#### HTTP-Based Service Discovery

**Use Cases**:
- Complex service mesh
- Custom discovery logic
- Advanced routing

### 6.2 Service Discovery Configuration

#### Create Service Discovery Service

```bash
aws servicediscovery create-service \
  --name web-service \
  --namespace-id ns-xxx \
  --dns-config "NamespaceId=ns-xxx,RoutingPolicy=WEIGHTED" \
  --health-check-custom-config "FailureThreshold=1"
```

#### Register with ECS Service

```json
{
  "serviceName": "web-service",
  "serviceRegistries": [
    {
      "registryArn": "arn:aws:servicediscovery:...:service/srv-xxx",
      "containerName": "web",
      "containerPort": 80
    }
  ]
}
```

---

## 7. Cost Optimization

### 7.1 Fargate Spot

**Savings**: Up to 70% discount compared to Fargate on-demand.

**Use Cases**:
- Fault-tolerant workloads
- Batch processing
- Development/testing
- Non-critical services

**Configuration**:
```json
{
  "capacityProviderStrategy": [
    {
      "capacityProvider": "FARGATE_SPOT",
      "weight": 70
    },
    {
      "capacityProvider": "FARGATE",
      "weight": 30,
      "base": 2
    }
  ]
}
```

### 7.2 Right-Sizing

**CPU and Memory**:
- Monitor actual usage
- Adjust task definitions
- Use CloudWatch metrics
- Avoid over-provisioning

**Example**:
```
Current: 2 vCPU, 4 GB memory
Actual Usage: 0.5 vCPU, 1 GB memory
Optimized: 0.5 vCPU, 1 GB memory
Savings: 75% cost reduction
```

### 7.3 Reserved Capacity

**Fargate**: No reserved capacity option (pay per use).

**EC2**: Use Reserved Instances for steady workloads.

**Savings Plans**: Flexible pricing for compute usage.

### 7.4 Auto Scaling

**Scale Down During Off-Hours**:
- Reduce desired count at night
- Scale up during business hours
- Use scheduled scaling

**Example Savings**:
```
24/7: 10 tasks × $0.05/hour × 24 hours = $12/day
Business Hours: 10 tasks × $0.05/hour × 8 hours = $4/day
Off-Hours: 2 tasks × $0.05/hour × 16 hours = $1.60/day
Total: $5.60/day
Savings: 53%
```

### 7.5 Resource Tagging

**Cost Allocation Tags**:
- Track costs by project
- Track costs by environment
- Track costs by team

**Tags**:
```json
{
  "tags": [
    {"key": "Project", "value": "WebApp"},
    {"key": "Environment", "value": "Production"},
    {"key": "Team", "value": "Backend"}
  ]
}
```

---

## 8. Troubleshooting

### 8.1 Common Issues

#### Tasks Not Starting

**Symptoms**:
- Tasks stuck in PENDING
- Service shows desired count not met

**Causes**:
- Insufficient CPU/memory
- No available capacity
- Security group issues
- Subnet issues

**Solutions**:
- Check CloudWatch metrics for capacity
- Verify security group rules
- Check subnet configuration
- Review task definition resources

#### Tasks Failing Health Checks

**Symptoms**:
- Tasks starting then stopping
- ALB marking targets unhealthy

**Causes**:
- Application not responding
- Wrong health check path
- Health check timeout too short
- Application startup time too long

**Solutions**:
- Increase `startPeriod` in health check
- Verify health check endpoint
- Check application logs
- Adjust health check timeout

#### High CPU/Memory Usage

**Symptoms**:
- Tasks being throttled
- Application slow
- Tasks being killed

**Causes**:
- Insufficient resources
- Memory leaks
- CPU-intensive operations

**Solutions**:
- Increase CPU/memory in task definition
- Optimize application code
- Add more tasks (horizontal scaling)
- Use larger instance types (EC2)

### 8.2 Debugging Tools

#### ECS Exec (Interactive Shell)

```bash
aws ecs execute-command \
  --cluster my-cluster \
  --task task-id \
  --container web \
  --interactive \
  --command "/bin/sh"
```

**Use Cases**:
- Debug running containers
- Check file system
- Test connectivity
- View environment variables

#### CloudWatch Logs Insights

**Query Logs**:
```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100
```

**Use Cases**:
- Find errors
- Analyze patterns
- Debug issues
- Monitor performance

#### ECS Service Events

**View Events**:
```bash
aws ecs describe-services \
  --cluster my-cluster \
  --services web-service \
  --query 'services[0].events[0:10]'
```

**Information**:
- Deployment status
- Task placement
- Health check results
- Error messages

### 8.3 Performance Tuning

#### Container Insights

**Enable for Performance Data**:
```bash
aws ecs update-cluster-settings \
  --cluster my-cluster \
  --settings name=containerInsights,value=enabled
```

**Metrics to Monitor**:
- CPU utilization
- Memory utilization
- Network I/O
- Storage I/O

#### Task Placement Optimization

**Spread Across AZs**:
```json
{
  "placementStrategy": [
    {"type": "spread", "field": "attribute:ecs.availability-zone"}
  ]
}
```

**Binpack for Efficiency**:
```json
{
  "placementStrategy": [
    {"type": "binpack", "field": "memory"}
  ]
}
```

---

## 9. Best Practices

### 9.1 Task Definition Best Practices

**Use Specific Image Tags**:
```json
// ❌ Bad
"image": "my-app:latest"

// ✅ Good
"image": "my-app:v1.2.3"
"image": "my-app:abc123def456"
```

**Set Resource Limits**:
```json
{
  "cpu": "512",
  "memory": "1024",
  "memoryReservation": "512"
}
```

**Use Health Checks**:
```json
{
  "healthCheck": {
    "command": ["CMD-SHELL", "curl -f http://localhost:80/health || exit 1"],
    "interval": 30,
    "timeout": 5,
    "retries": 3,
    "startPeriod": 60
  }
}
```

**Use Secrets, Not Environment Variables**:
```json
// ❌ Bad
"environment": [
  {"name": "DB_PASSWORD", "value": "secret123"}
]

// ✅ Good
"secrets": [
  {
    "name": "DB_PASSWORD",
    "valueFrom": "arn:aws:secretsmanager:...:secret:db-password"
  }
]
```

### 9.2 Service Best Practices

**Use Deployment Circuit Breaker**:
```json
{
  "deploymentConfiguration": {
    "deploymentCircuitBreaker": {
      "enable": true,
      "rollback": true
    }
  }
}
```

**Configure Health Check Grace Period**:
```json
{
  "healthCheckGracePeriodSeconds": 60
}
```

**Use Multiple AZs**:
```json
{
  "networkConfiguration": {
    "awsvpcConfiguration": {
      "subnets": ["subnet-az1", "subnet-az2", "subnet-az3"]
    }
  }
}
```

### 9.3 Security Best Practices

**Least Privilege IAM Roles**:
- Grant minimum permissions
- Use separate roles for execution and task
- Regularly review permissions

**Encrypt Data**:
- Encrypt data at rest (EBS, EFS)
- Encrypt data in transit (TLS)
- Use KMS for key management

**Network Isolation**:
- Use private subnets
- Use security groups
- Use VPC endpoints
- Minimize public exposure

**Image Security**:
- Scan images for vulnerabilities
- Use trusted base images
- Keep images updated
- Use immutable tags

### 9.4 Monitoring Best Practices

**Enable Container Insights**:
- Automatic dashboards
- Performance metrics
- Anomaly detection

**Set Up Alarms**:
- CPU utilization
- Memory utilization
- Task failures
- Service health

**Centralized Logging**:
- Use CloudWatch Logs
- Set retention policies
- Use log groups per service
- Enable log encryption

### 9.5 Cost Optimization Best Practices

**Right-Size Resources**:
- Monitor actual usage
- Adjust CPU/memory
- Use CloudWatch metrics

**Use Fargate Spot**:
- For fault-tolerant workloads
- Up to 70% savings
- Mix with on-demand

**Auto Scaling**:
- Scale down during off-hours
- Use scheduled scaling
- Optimize min/max capacity

**Resource Tagging**:
- Tag all resources
- Use cost allocation tags
- Track spending by project

---

## 10. Real-World Examples

### 10.1 Web Application (Fargate + ALB)

**Architecture**:
```
Internet
    │
    ▼
ALB (Application Load Balancer)
    │
    ├─→ Fargate Task 1 (web:80)
    ├─→ Fargate Task 2 (web:80)
    └─→ Fargate Task 3 (web:80)
```

**Configuration**:
- Launch Type: Fargate
- Network Mode: awsvpc
- Load Balancer: ALB
- Auto Scaling: CPU-based
- Health Checks: Container + ALB

### 10.2 Microservices (ECS + Service Discovery)

**Architecture**:
```
API Gateway
    │
    ├─→ User Service (ECS)
    ├─→ Order Service (ECS)
    └─→ Payment Service (ECS)
        │
        └─→ (Service Discovery)
            └─→ Database Service (ECS)
```

**Configuration**:
- Service Discovery: Cloud Map
- Inter-service communication: DNS-based
- Load Balancing: Internal ALB
- Monitoring: X-Ray tracing

### 10.3 Batch Processing (EC2 + Spot)

**Architecture**:
```
S3 (Input Data)
    │
    ▼
ECS Batch Job (EC2 Spot)
    │
    ├─→ Process Data
    └─→
S3 (Output Data)
```

**Configuration**:
- Launch Type: EC2
- Instance Type: Spot instances
- Auto Scaling: Based on queue depth
- Cost: 70-90% savings

### 10.4 High-Performance Application (EC2 + NLB)

**Architecture**:
```
Internet
    │
    ▼
NLB (Network Load Balancer)
    │
    ├─→ EC2 Instance 1 (Multiple Tasks)
    ├─→ EC2 Instance 2 (Multiple Tasks)
    └─→ EC2 Instance 3 (Multiple Tasks)
```

**Configuration**:
- Launch Type: EC2
- Load Balancer: NLB
- Network Mode: bridge (for multiple tasks)
- Instance Type: Compute-optimized (c5)

---

## Summary of Part 2

**Key Takeaways**:
1. **Load Balancing**: ALB for HTTP/HTTPS, NLB for TCP/UDP, GWLB for network appliances
2. **Auto Scaling**: Service scaling (tasks) and cluster scaling (instances)
3. **Deployment Strategies**: Rolling, blue/green, canary, circuit breaker
4. **Monitoring**: CloudWatch metrics, logs, Container Insights, X-Ray
5. **Security**: IAM roles, secrets management, network security, container security
6. **Service Discovery**: Cloud Map for DNS-based service discovery
7. **Cost Optimization**: Fargate Spot, right-sizing, auto-scaling, tagging
8. **Troubleshooting**: Common issues, debugging tools, performance tuning
9. **Best Practices**: Task definitions, services, security, monitoring, cost
10. **Real-World Examples**: Web apps, microservices, batch processing, high-performance apps

**Complete ECS Knowledge**:
- Part 1: Fundamentals, architecture, launch types, task definitions, networking
- Part 2: Advanced topics, operations, best practices, real-world examples

This comprehensive guide covers everything you need to know about AWS ECS, from basics to advanced production deployments!

