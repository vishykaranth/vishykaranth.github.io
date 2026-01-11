# AWS EC2 In-Depth Interview Guide - Part 3: Scaling, Monitoring & Advanced Topics

## Table of Contents
1. [Auto Scaling](#auto-scaling)
2. [Elastic Load Balancing](#elastic-load-balancing)
3. [Monitoring & Logging](#monitoring--logging)
4. [CloudWatch](#cloudwatch)
5. [Systems Manager](#systems-manager)
6. [Best Practices](#best-practices)
7. [Advanced Topics](#advanced-topics)
8. [Troubleshooting](#troubleshooting)
9. [Real-World Scenarios](#real-world-scenarios)

---

## Auto Scaling

### What is Auto Scaling?

**Auto Scaling** automatically adjusts the number of EC2 instances based on:
- **Demand**: Scale up during high traffic, scale down during low traffic
- **Cost Optimization**: Run only needed instances
- **Availability**: Maintain minimum healthy instances
- **Performance**: Ensure adequate capacity

### Auto Scaling Components

1. **Launch Template/Configuration**: Defines instance specifications
2. **Auto Scaling Group (ASG)**: Group of instances managed together
3. **Scaling Policies**: Rules for when to scale
4. **Health Checks**: Monitor instance health
5. **Lifecycle Hooks**: Actions during instance lifecycle

### Launch Templates vs Launch Configurations

| Feature | Launch Template | Launch Configuration |
|---------|----------------|---------------------|
| Versioning | Yes (multiple versions) | No |
| Parameter Validation | Yes | No |
| Instance Types | Multiple (mixed instances) | Single |
| Spot Instances | Yes (mixed instances) | Limited |
| Recommended | Yes (modern) | Legacy |

### Hands-On: Creating Launch Template

```bash
# Create launch template
aws ec2 create-launch-template \
  --launch-template-name my-web-server-template \
  --version-description "Initial version" \
  --launch-template-data '{
    "ImageId": "ami-0c55b159cbfafe1f0",
    "InstanceType": "t3.micro",
    "KeyName": "my-key-pair",
    "SecurityGroupIds": ["sg-0123456789abcdef0"],
    "UserData": "IyEvYmluL2Jhc2gKeXVtIHVwZGF0ZSAteQp5dW0gaW5zdGFsbCAteSBodHRwZApzeXN0ZW1jdGwgc3RhcnQgaHR0cGQKc3lzdGVtY3RsIGVuYWJsZSBodHRwZAplY2hvICI8aDE+SGVsbG8gZnJvbSBBdXRvIFNjYWxpbmc8L2gxPiIgPiAvdmFyL3d3dy9odG1sL2luZGV4Lmh0bWw=",
    "IamInstanceProfile": {
      "Name": "EC2-S3-Access-Profile"
    },
    "TagSpecifications": [{
      "ResourceType": "instance",
      "Tags": [{"Key": "Name", "Value": "WebServer"}]
    }]
  }'

# Create launch template version
aws ec2 create-launch-template-version \
  --launch-template-name my-web-server-template \
  --source-version 1 \
  --version-description "Updated version" \
  --launch-template-data '{
    "InstanceType": "t3.small"
  }'
```

### Hands-On: Creating Auto Scaling Group

```bash
# Create Auto Scaling Group
aws autoscaling create-auto-scaling-group \
  --auto-scaling-group-name my-web-asg \
  --launch-template LaunchTemplateName=my-web-server-template,Version='$Latest' \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 2 \
  --vpc-zone-identifier "subnet-0123456789abcdef0,subnet-0987654321fedcba0" \
  --health-check-type ELB \
  --health-check-grace-period 300 \
  --target-group-arns arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/my-target-group/1234567890123456 \
  --tags ResourceId=my-web-asg,ResourceType=auto-scaling-group,Key=Name,Value=WebASG,PropagateAtLaunch=true
```

### Scaling Policies

#### 1. **Target Tracking Scaling**

**Simplest scaling policy** - Maintains a target metric value

```bash
# Create target tracking scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-web-asg \
  --policy-name cpu-target-tracking \
  --policy-type TargetTrackingScaling \
  --target-tracking-configuration '{
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 70.0,
    "ScaleInCooldown": 300,
    "ScaleOutCooldown": 60
  }'
```

#### 2. **Step Scaling**

**Scales based on CloudWatch alarms** with multiple steps

```bash
# Create CloudWatch alarm for high CPU
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu-alarm \
  --alarm-description "Alarm when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:autoscaling:us-east-1:123456789012:scalingPolicy:policy-id

# Create step scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-web-asg \
  --policy-name step-scaling-policy \
  --policy-type StepScaling \
  --adjustment-type ChangeInCapacity \
  --step-adjustments '[
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
  ]' \
  --cooldown 300
```

#### 3. **Simple Scaling**

**Scales by fixed amount** based on CloudWatch alarm

```bash
# Create simple scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-web-asg \
  --policy-name simple-scaling-policy \
  --policy-type SimpleScaling \
  --adjustment-type ChangeInCapacity \
  --scaling-adjustment 2 \
  --cooldown 300
```

### Scheduled Scaling

**Scale at specific times** (e.g., business hours)

```bash
# Create scheduled action for business hours
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name my-web-asg \
  --scheduled-action-name scale-up-business-hours \
  --start-time "2024-01-01T09:00:00Z" \
  --recurrence "0 9 * * MON-FRI" \
  --min-size 5 \
  --max-size 15 \
  --desired-capacity 5

# Create scheduled action for off-hours
aws autoscaling put-scheduled-update-group-action \
  --auto-scaling-group-name my-web-asg \
  --scheduled-action-name scale-down-off-hours \
  --start-time "2024-01-01T18:00:00Z" \
  --recurrence "0 18 * * MON-FRI" \
  --min-size 2 \
  --max-size 10 \
  --desired-capacity 2
```

### Predictive Scaling

**Uses machine learning** to predict traffic and scale proactively

```bash
# Create predictive scaling policy
aws autoscaling put-scaling-policy \
  --auto-scaling-group-name my-web-asg \
  --policy-name predictive-scaling-policy \
  --policy-type PredictiveScaling \
  --predictive-scaling-configuration '{
    "MetricSpecifications": [{
      "TargetValue": 70.0,
      "PredefinedMetricPairSpecification": {
        "PredefinedMetricType": "ASGCPUUtilization"
      }
    }],
    "Mode": "ForecastAndScale",
    "SchedulingBufferTime": 10
  }'
```

### Mixed Instances Policy

**Use multiple instance types** for cost optimization and availability

```bash
# Update ASG with mixed instances policy
aws autoscaling update-auto-scaling-group \
  --auto-scaling-group-name my-web-asg \
  --mixed-instances-policy '{
    "LaunchTemplate": {
      "LaunchTemplateSpecification": {
        "LaunchTemplateName": "my-web-server-template",
        "Version": "$Latest"
      },
      "Overrides": [
        {"InstanceType": "t3.micro"},
        {"InstanceType": "t3.small"},
        {"InstanceType": "t3.medium"}
      ]
    },
    "InstancesDistribution": {
      "OnDemandPercentageAboveBaseCapacity": 0,
      "SpotInstancePools": 2,
      "SpotAllocationStrategy": "lowest-price"
    }
  }'
```

### Lifecycle Hooks

**Execute custom actions** during instance lifecycle

```bash
# Create lifecycle hook for instance launch
aws autoscaling put-lifecycle-hook \
  --lifecycle-hook-name instance-launch-hook \
  --auto-scaling-group-name my-web-asg \
  --lifecycle-transition autoscaling:EC2_INSTANCE_LAUNCHING \
  --heartbeat-timeout 300 \
  --default-result CONTINUE

# Create lifecycle hook for instance termination
aws autoscaling put-lifecycle-hook \
  --lifecycle-hook-name instance-terminate-hook \
  --auto-scaling-group-name my-web-asg \
  --lifecycle-transition autoscaling:EC2_INSTANCE_TERMINATING \
  --heartbeat-timeout 300 \
  --default-result CONTINUE
```

### Auto Scaling Best Practices

1. **Use Launch Templates**: Modern, versioned, supports mixed instances
2. **Multiple AZs**: Distribute instances across availability zones
3. **Health Checks**: Use ELB health checks for better reliability
4. **Cooldown Periods**: Prevent rapid scaling oscillations
5. **Predictive Scaling**: Use for predictable workloads
6. **Mixed Instances**: Combine On-Demand and Spot for cost savings
7. **Lifecycle Hooks**: For graceful shutdowns and custom initialization

---

## Elastic Load Balancing

### What is ELB?

**Elastic Load Balancing** distributes incoming traffic across multiple targets:
- **High Availability**: Distributes traffic across healthy instances
- **Fault Tolerance**: Automatically routes away from unhealthy instances
- **SSL/TLS Termination**: Handles SSL certificates
- **Health Checks**: Monitors target health

### Load Balancer Types

#### 1. **Application Load Balancer (ALB)**

**Layer 7 (HTTP/HTTPS)**
- **Features**: Path-based, host-based routing, container support
- **Use Cases**: Web applications, microservices, containerized apps
- **Ports**: 80, 443, 8080
- **Targets**: EC2, IP addresses, Lambda functions, containers

**Hands-On: Create ALB**

```bash
# Create ALB
aws elbv2 create-load-balancer \
  --name my-web-alb \
  --subnets subnet-0123456789abcdef0 subnet-0987654321fedcba0 \
  --security-groups sg-0123456789abcdef0 \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4 \
  --tags Key=Name,Value=WebALB

# Create target group
aws elbv2 create-target-group \
  --name web-targets \
  --protocol HTTP \
  --port 80 \
  --vpc-id vpc-0123456789abcdef0 \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --target-type instance

# Register targets
aws elbv2 register-targets \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/web-targets/1234567890123456 \
  --targets Id=i-1234567890abcdef0 Id=i-0987654321fedcba0

# Create listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-web-alb/1234567890123456 \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/web-targets/1234567890123456
```

**Path-Based Routing**

```bash
# Create listener rule for path-based routing
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:listener/app/my-web-alb/1234567890123456/abcdef1234567890 \
  --priority 100 \
  --conditions Field=path-pattern,Values='/api/*' \
  --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/api-targets/1234567890123456
```

**Host-Based Routing**

```bash
# Create listener rule for host-based routing
aws elbv2 create-rule \
  --listener-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:listener/app/my-web-alb/1234567890123456/abcdef1234567890 \
  --priority 200 \
  --conditions Field=host-header,Values='api.example.com' \
  --actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/api-targets/1234567890123456
```

#### 2. **Network Load Balancer (NLB)**

**Layer 4 (TCP/UDP)**
- **Features**: Ultra-low latency, high throughput, static IP
- **Use Cases**: TCP/UDP traffic, extreme performance, static IP requirements
- **Ports**: Any port
- **Targets**: EC2, IP addresses, Application Load Balancers

**Hands-On: Create NLB**

```bash
# Create NLB
aws elbv2 create-load-balancer \
  --name my-network-nlb \
  --subnets subnet-0123456789abcdef0 subnet-0987654321fedcba0 \
  --type network \
  --scheme internet-facing \
  --ip-address-type ipv4

# Create target group for NLB
aws elbv2 create-target-group \
  --name nlb-targets \
  --protocol TCP \
  --port 80 \
  --vpc-id vpc-0123456789abcdef0 \
  --target-type instance \
  --health-check-protocol TCP \
  --health-check-port 80 \
  --health-check-interval-seconds 30 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3
```

#### 3. **Classic Load Balancer (CLB)**

**Legacy (Layer 4/7)**
- **Features**: Basic load balancing, EC2-Classic support
- **Use Cases**: Legacy applications, EC2-Classic
- **Not Recommended**: Use ALB or NLB for new applications

### Load Balancer Comparison

| Feature | ALB | NLB | CLB |
|---------|-----|-----|-----|
| Layer | 7 (HTTP/HTTPS) | 4 (TCP/UDP) | 4/7 |
| Path-Based Routing | Yes | No | No |
| Host-Based Routing | Yes | No | No |
| Static IP | No | Yes | No |
| Lambda Targets | Yes | No | No |
| Container Support | Yes | Yes | No |
| Connection Draining | Yes | Yes | Yes |
| SSL/TLS Termination | Yes | Yes | Yes |

### Health Checks

**Health Check Configuration**

```bash
# Update target group health check
aws elbv2 modify-target-group \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/web-targets/1234567890123456 \
  --health-check-path /health \
  --health-check-interval-seconds 30 \
  --health-check-timeout-seconds 5 \
  --healthy-threshold-count 2 \
  --unhealthy-threshold-count 3 \
  --health-check-protocol HTTP \
  --health-check-port 80 \
  --matcher HttpCode=200
```

**Health Check States:**
- **Healthy**: Target responds to health checks
- **Unhealthy**: Target fails health checks
- **Initial**: Target is being checked
- **Draining**: Target is being deregistered

### SSL/TLS Termination

```bash
# Import SSL certificate to ACM
aws acm import-certificate \
  --certificate fileb://certificate.pem \
  --private-key fileb://private-key.pem \
  --certificate-chain fileb://certificate-chain.pem \
  --region us-east-1

# Create HTTPS listener
aws elbv2 create-listener \
  --load-balancer-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:loadbalancer/app/my-web-alb/1234567890123456 \
  --protocol HTTPS \
  --port 443 \
  --certificates CertificateArn=arn:aws:acm:us-east-1:123456789012:certificate/12345678-1234-1234-1234-123456789012 \
  --default-actions Type=forward,TargetGroupArn=arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/web-targets/1234567890123456 \
  --ssl-policy ELBSecurityPolicy-TLS-1-2-2017-01
```

### Connection Draining (Deregistration Delay)

```bash
# Enable connection draining
aws elbv2 modify-target-group-attributes \
  --target-group-arn arn:aws:elasticloadbalancing:us-east-1:123456789012:targetgroup/web-targets/1234567890123456 \
  --attributes Key=deregistration_delay.timeout_seconds,Value=300
```

---

## Monitoring & Logging

### CloudWatch Metrics

**EC2 Metrics** (automatically collected):
- `CPUUtilization`: CPU usage percentage
- `NetworkIn/NetworkOut`: Network traffic
- `DiskReadOps/DiskWriteOps`: Disk I/O operations
- `StatusCheckFailed`: Instance status check failures
- `StatusCheckFailed_Instance`: Instance-level failures
- `StatusCheckFailed_System`: System-level failures

**Custom Metrics**

```bash
# Put custom metric
aws cloudwatch put-metric-data \
  --namespace MyApp/Metrics \
  --metric-name ActiveUsers \
  --value 150 \
  --unit Count \
  --timestamp $(date -u +"%Y-%m-%dT%H:%M:%S")

# Put metric with dimensions
aws cloudwatch put-metric-data \
  --namespace MyApp/Metrics \
  --metric-name RequestLatency \
  --value 250 \
  --unit Milliseconds \
  --dimensions Environment=Production,Server=WebServer1
```

### CloudWatch Alarms

```bash
# Create alarm for high CPU
aws cloudwatch put-metric-alarm \
  --alarm-name high-cpu-alarm \
  --alarm-description "Alarm when CPU exceeds 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts-topic \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0

# Create alarm for instance status check
aws cloudwatch put-metric-alarm \
  --alarm-name instance-status-check-failed \
  --alarm-description "Alarm when instance status check fails" \
  --metric-name StatusCheckFailed \
  --namespace AWS/EC2 \
  --statistic Maximum \
  --period 60 \
  --threshold 1 \
  --comparison-operator GreaterThanOrEqualToThreshold \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts-topic \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0
```

### CloudWatch Logs

**EC2 Instance Logs**

```bash
# Create log group
aws logs create-log-group \
  --log-group-name /aws/ec2/my-app \
  --retention-in-days 7

# Create log stream
aws logs create-log-stream \
  --log-group-name /aws/ec2/my-app \
  --log-stream-name instance-i-1234567890abcdef0

# Put log events
aws logs put-log-events \
  --log-group-name /aws/ec2/my-app \
  --log-stream-name instance-i-1234567890abcdef0 \
  --log-events timestamp=$(date +%s)000,message="Application started"
```

**CloudWatch Logs Agent Configuration**

```json
{
  "agent": {
    "metrics_collection_interval": 60
  },
  "logs": {
    "logs_collected": {
      "files": {
        "collect_list": [
          {
            "file_path": "/var/log/application.log",
            "log_group_name": "/aws/ec2/my-app",
            "log_stream_name": "{instance_id}",
            "timezone": "UTC"
          }
        ]
      }
    }
  }
}
```

### CloudWatch Insights

**Query Logs with CloudWatch Insights**

```bash
# Query logs
aws logs start-query \
  --log-group-name /aws/ec2/my-app \
  --start-time $(date -d '1 hour ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | sort @timestamp desc | limit 100'
```

---

## Systems Manager

### What is Systems Manager?

**AWS Systems Manager** provides:
- **Patch Management**: Automate patching
- **Parameter Store**: Secure configuration storage
- **Session Manager**: Secure shell access without SSH
- **Run Command**: Execute commands on instances
- **State Manager**: Maintain instance state

### Session Manager (SSH Alternative)

**Benefits:**
- No SSH keys needed
- No open ports (22)
- Centralized logging
- IAM-based access control

```bash
# Start session
aws ssm start-session \
  --target i-1234567890abcdef0

# Execute command
aws ssm send-command \
  --instance-ids i-1234567890abcdef0 \
  --document-name "AWS-RunShellScript" \
  --parameters 'commands=["yum update -y"]' \
  --output-s3-bucket-name my-command-output \
  --output-s3-key-prefix commands/
```

### Parameter Store

**Store Configuration Securely**

```bash
# Store parameter
aws ssm put-parameter \
  --name /myapp/database/host \
  --value "db.example.com" \
  --type String \
  --description "Database hostname"

# Store secure parameter
aws ssm put-parameter \
  --name /myapp/database/password \
  --value "secretpassword" \
  --type SecureString \
  --description "Database password" \
  --key-id alias/aws/ssm

# Get parameter
aws ssm get-parameter \
  --name /myapp/database/host \
  --with-decryption

# Get multiple parameters
aws ssm get-parameters \
  --names /myapp/database/host /myapp/database/password \
  --with-decryption
```

### Patch Manager

```bash
# Create patch baseline
aws ssm create-patch-baseline \
  --name Production-Baseline \
  --description "Production patch baseline" \
  --operating-system AMAZON_LINUX_2 \
  --approval-rules '{
    "PatchRules": [{
      "PatchFilterGroup": {
        "PatchFilters": [{
          "Key": "CLASSIFICATION",
          "Values": ["SecurityUpdates", "CriticalUpdates"]
        }]
      },
      "ComplianceLevel": "CRITICAL",
      "ApproveAfterDays": 7
    }]
  }'
```

---

## Best Practices

### Security Best Practices

1. **Use Security Groups**: Restrict access to necessary ports only
2. **IAM Roles**: Use IAM roles instead of access keys
3. **Encryption**: Encrypt EBS volumes and snapshots
4. **Key Pairs**: Rotate key pairs regularly
5. **VPC**: Use private subnets for databases
6. **Patch Management**: Keep instances updated
7. **Least Privilege**: Grant minimum necessary permissions
8. **Multi-Factor Authentication**: Enable MFA for console access

### Cost Optimization

1. **Right-Sizing**: Match instance type to workload
2. **Reserved Instances**: For steady-state workloads
3. **Spot Instances**: For fault-tolerant workloads
4. **Auto Scaling**: Scale down during low usage
5. **Terminate Unused**: Clean up development instances
6. **EBS Optimization**: Use appropriate volume types
7. **Monitoring**: Track costs with Cost Explorer

### High Availability

1. **Multiple AZs**: Distribute across availability zones
2. **Auto Scaling**: Maintain minimum healthy instances
3. **Load Balancing**: Distribute traffic
4. **Health Checks**: Monitor instance health
5. **Backup Strategy**: Regular EBS snapshots
6. **Disaster Recovery**: Multi-region deployment

### Performance Optimization

1. **Instance Types**: Choose appropriate instance family
2. **EBS Optimization**: Use EBS-optimized instances
3. **Enhanced Networking**: Use SR-IOV for high performance
4. **Placement Groups**: Use for low-latency requirements
5. **Caching**: Implement application-level caching
6. **CDN**: Use CloudFront for static content

---

## Advanced Topics

### Enhanced Networking

**SR-IOV (Single Root I/O Virtualization)**
- Higher network performance
- Lower latency
- Lower jitter
- Available on most instance types

```bash
# Check if instance supports enhanced networking
aws ec2 describe-instance-types \
  --instance-types m5.large \
  --query 'InstanceTypes[0].NetworkInfo.EnaSupport'

# Enable enhanced networking (requires instance stop)
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --ena-support
```

### EBS-Optimized Instances

**Dedicated bandwidth** for EBS volumes

```bash
# Check if instance is EBS-optimized
aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[0].Instances[0].EbsOptimized'

# Enable EBS optimization (requires instance stop)
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --ebs-optimized
```

### Dedicated Instances vs Dedicated Hosts

**Dedicated Instances:**
- Hardware dedicated to your account
- May share hardware with other instances in your account
- No control over instance placement
- Additional per-hour charge

**Dedicated Hosts:**
- Physical server dedicated to you
- Full control over instance placement
- Can use existing server-bound licenses
- Most expensive option

### Instance Metadata Service (IMDS)

**Access instance metadata**

```bash
# From within instance
curl http://169.254.169.254/latest/meta-data/

# Get instance ID
curl http://169.254.169.254/latest/meta-data/instance-id

# Get IAM role credentials (IMDSv2 - requires token)
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/

# Get user data
curl http://169.254.169.254/latest/user-data
```

### Nitro Enclaves

**Isolated compute environments** for sensitive workloads

```bash
# Launch instance with enclave support
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type m5.large \
  --enclave-options Enabled=true \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0
```

---

## Troubleshooting

### Common Issues

#### 1. **Instance Not Accessible via SSH**

**Check:**
- Security group allows port 22
- Key pair is correct
- Instance is in running state
- Public IP is assigned (if needed)
- Network ACLs allow traffic

```bash
# Check instance state
aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[0].Instances[0].[State.Name,PublicIpAddress,PrivateIpAddress]'

# Check security groups
aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[0].Instances[0].SecurityGroups'

# Check security group rules
aws ec2 describe-security-groups \
  --group-ids sg-0123456789abcdef0
```

#### 2. **High CPU Utilization**

**Solutions:**
- Right-size instance type
- Implement auto scaling
- Optimize application code
- Use CloudWatch alarms

```bash
# Get CPU metrics
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-1234567890abcdef0 \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%S) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%S) \
  --period 300 \
  --statistics Average
```

#### 3. **EBS Volume Performance Issues**

**Solutions:**
- Check volume type (gp3 vs io2)
- Increase IOPS (for gp3/io2)
- Use EBS-optimized instances
- Check instance network performance

```bash
# Check volume type and IOPS
aws ec2 describe-volumes \
  --volume-ids vol-0123456789abcdef0 \
  --query 'Volumes[0].[VolumeType,Iops,Throughput]'

# Modify volume IOPS
aws ec2 modify-volume \
  --volume-id vol-0123456789abcdef0 \
  --iops 10000
```

#### 4. **Instance Status Check Failures**

**System Status Check Failure:**
- Hardware/network issue
- AWS will migrate instance automatically

**Instance Status Check Failure:**
- Application/OS issue
- Requires manual intervention

```bash
# Check status checks
aws ec2 describe-instance-status \
  --instance-ids i-1234567890abcdef0 \
  --include-all-instances

# Reboot instance
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0
```

---

## Real-World Scenarios

### Scenario 1: High-Traffic Web Application

**Requirements:**
- Handle variable traffic (100-10,000 requests/second)
- 99.9% availability
- Low latency (<100ms)
- Cost-effective

**Solution:**
```
1. Application Load Balancer (ALB)
   - Path-based routing
   - SSL/TLS termination
   - Health checks

2. Auto Scaling Group
   - Launch template with AMI
   - Min: 2, Max: 50, Desired: 5
   - Target tracking: 70% CPU
   - Multiple AZs

3. Instance Configuration
   - Instance type: t3.medium (start)
   - Mixed instances: t3.medium, t3.large
   - Spot instances: 50% of capacity

4. Monitoring
   - CloudWatch alarms
   - Custom metrics
   - Log aggregation

5. Storage
   - EBS gp3 for boot volumes
   - S3 for static assets
   - CloudFront CDN
```

### Scenario 2: Database Server

**Requirements:**
- High IOPS (50,000+)
- Persistent storage
- Backup strategy
- Encryption

**Solution:**
```
1. Instance Type
   - r5.2xlarge (memory-optimized)
   - EBS-optimized
   - Enhanced networking

2. Storage
   - io2 volume: 500 GB
   - IOPS: 50,000
   - Multi-Attach enabled (if needed)

3. Backup
   - Automated EBS snapshots
   - Daily snapshots, 30-day retention
   - Cross-region snapshot copy

4. Security
   - Private subnet
   - Security group: Only from app servers
   - Encrypted volumes
   - IAM role for S3 backup access

5. High Availability
   - Multi-AZ deployment
   - Read replicas
   - Automated failover
```

### Scenario 3: Batch Processing

**Requirements:**
- Process large datasets
- Cost-effective
- Can tolerate interruptions
- Parallel processing

**Solution:**
```
1. Spot Instances
   - Spot Fleet with multiple instance types
   - Diversification strategy
   - Auto-replacement on interruption

2. Auto Scaling
   - Scale based on queue depth
   - Use SQS for job queue
   - CloudWatch metric: ApproximateNumberOfMessages

3. Storage
   - S3 for input/output data
   - Instance store for temporary processing
   - EBS for checkpoint data

4. Monitoring
   - CloudWatch metrics
   - SNS notifications for failures
   - CloudWatch Logs for debugging
```

---

## Interview Questions & Answers

### Q1: How does Auto Scaling work with Load Balancers?

**Answer:**
- Auto Scaling Group registers instances with target groups
- Load balancer performs health checks on targets
- Unhealthy targets are marked as unhealthy
- Auto Scaling replaces unhealthy instances
- Load balancer distributes traffic only to healthy targets

### Q2: What's the difference between ALB and NLB?

**Answer:**
- **ALB**: Layer 7, path/host-based routing, supports Lambda, container-aware
- **NLB**: Layer 4, ultra-low latency, static IP, high throughput, TCP/UDP only

### Q3: How do you handle SSL/TLS termination?

**Answer:**
- **ALB/NLB**: Terminate SSL at load balancer (recommended)
- **EC2 Instance**: Terminate SSL on instance (more CPU usage)
- Use ACM for certificate management
- Offloads SSL processing from instances

### Q4: What happens when an Auto Scaling instance fails health checks?

**Answer:**
1. Load balancer marks target as unhealthy
2. Auto Scaling detects unhealthy instance
3. After grace period, instance is terminated
4. New instance is launched to replace it
5. New instance passes health checks and receives traffic

### Q5: How do you implement blue-green deployment with EC2?

**Answer:**
1. Create new Auto Scaling Group with new AMI
2. Register new instances with target group
3. Gradually shift traffic (weighted target groups)
4. Monitor new instances
5. Complete cutover or rollback
6. Terminate old instances

### Q6: What's the difference between On-Demand and Spot Instances in Auto Scaling?

**Answer:**
- **On-Demand**: Guaranteed availability, higher cost, no interruption
- **Spot**: Up to 90% discount, can be interrupted, best for fault-tolerant workloads
- **Mixed Instances**: Use both for cost optimization and availability

---

## Summary

**Key Takeaways:**
1. **Auto Scaling**: Automatically adjust capacity based on demand
2. **Load Balancing**: Distribute traffic across healthy instances
3. **Monitoring**: CloudWatch for metrics, alarms, and logs
4. **Systems Manager**: Patch management, parameter store, session manager
5. **Best Practices**: Security, cost optimization, high availability
6. **Advanced Topics**: Enhanced networking, EBS optimization, metadata service
7. **Troubleshooting**: Common issues and solutions
8. **Real-World Scenarios**: Practical implementation patterns

**Complete EC2 Guide:**
- **Part 1**: Fundamentals, Instance Types, AMIs, Pricing
- **Part 2**: Storage, Networking, Security Groups, Key Pairs
- **Part 3**: Auto Scaling, Load Balancing, Monitoring, Advanced Topics

---

**Part 3 Complete** - All 3 parts ready for interview preparation!

