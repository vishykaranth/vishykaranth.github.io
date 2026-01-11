# AWS EC2 (Elastic Compute Cloud) - In-Depth Guide

## Part 2: Advanced Topics, Auto Scaling, Monitoring, and Best Practices

---

## Table of Contents

1. [Auto Scaling](#1-auto-scaling)
2. [Load Balancing](#2-load-balancing)
3. [Placement Groups](#3-placement-groups)
4. [Elastic IPs and ENIs](#4-elastic-ips-and-enis)
5. [IAM Roles for EC2](#5-iam-roles-for-ec2)
6. [Monitoring and Logging](#6-monitoring-and-logging)
7. [EC2 Instance Metadata](#7-ec2-instance-metadata)
8. [Cost Optimization](#8-cost-optimization)
9. [High Availability and Disaster Recovery](#9-high-availability-and-disaster-recovery)
10. [Best Practices](#10-best-practices)
11. [Troubleshooting](#11-troubleshooting)

---

## 1. Auto Scaling

### 1.1 What is Auto Scaling?

**Auto Scaling** automatically adjusts the number of EC2 instances based on demand, ensuring you have the right amount of capacity to handle your application load.

### 1.2 Auto Scaling Components

#### Launch Template / Launch Configuration
- Template for launching instances
- Defines AMI, instance type, security groups, user data, etc.
- Launch Template (newer, more flexible) vs Launch Configuration (older)

#### Auto Scaling Group (ASG)
- Collection of EC2 instances managed together
- Defines min, max, and desired capacity
- Distributes instances across Availability Zones
- Handles health checks and replacements

#### Scaling Policies
- Define when and how to scale
- Types: Target Tracking, Step Scaling, Simple Scaling, Scheduled Scaling

### 1.3 Auto Scaling Group Configuration

**Basic Configuration**:
```json
{
  "MinSize": 2,
  "MaxSize": 10,
  "DesiredCapacity": 4,
  "AvailabilityZones": ["us-east-1a", "us-east-1b"],
  "HealthCheckType": "ELB",
  "HealthCheckGracePeriod": 300
}
```

**Key Parameters**:
- **Min Size**: Minimum number of instances (always running)
- **Max Size**: Maximum number of instances (scaling limit)
- **Desired Capacity**: Target number of instances
- **Health Check**: EC2 or ELB health checks
- **Termination Policies**: Which instances to terminate when scaling in

### 1.4 Scaling Policies

#### Target Tracking Scaling
**Use Case**: Maintain a specific metric at a target value

**Example**:
```json
{
  "PolicyType": "TargetTrackingScaling",
  "TargetValue": 70.0,
  "PredefinedMetricSpecification": {
    "PredefinedMetricType": "ASGAverageCPUUtilization"
  }
}
```

**Behavior**: Keeps average CPU utilization at 70%
- CPU > 70% → Add instances
- CPU < 70% → Remove instances

#### Step Scaling
**Use Case**: Scale based on specific metric thresholds

**Example**:
```json
{
  "PolicyType": "StepScaling",
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
    }
  ],
  "AlarmThreshold": 70
}
```

**Behavior**:
- CPU 70-80% → Add 1 instance
- CPU 80-90% → Add 2 instances
- CPU > 90% → Add 3 instances

#### Simple Scaling
**Use Case**: Simple scale up/down by fixed amount

**Example**:
```json
{
  "PolicyType": "SimpleScaling",
  "ScalingAdjustment": 2,
  "Cooldown": 300
}
```

**Behavior**: Add/remove 2 instances, wait 5 minutes before next action

#### Scheduled Scaling
**Use Case**: Scale at specific times

**Example**:
```json
{
  "ScheduledActionName": "ScaleUpMorning",
  "Schedule": "0 9 * * *",
  "DesiredCapacity": 10
}
```

**Behavior**: Scale to 10 instances every day at 9 AM

### 1.5 Auto Scaling Best Practices

**Health Checks**:
- Use ELB health checks for application-level health
- Set appropriate grace period
- Monitor unhealthy instances

**Scaling Policies**:
- Use target tracking for simple scenarios
- Use step scaling for complex scenarios
- Set appropriate cooldown periods

**Capacity Planning**:
- Set realistic min/max values
- Consider cost implications
- Test scaling behavior

**Example Auto Scaling Setup**:
```bash
# Create Launch Template
aws ec2 create-launch-template \
    --launch-template-name web-server-template \
    --launch-template-data '{
        "ImageId": "ami-0c55b159cbfafe1f0",
        "InstanceType": "t3.medium",
        "SecurityGroupIds": ["sg-12345678"],
        "UserData": "base64-encoded-script"
    }'

# Create Auto Scaling Group
aws autoscaling create-auto-scaling-group \
    --auto-scaling-group-name web-servers-asg \
    --launch-template LaunchTemplateName=web-server-template \
    --min-size 2 \
    --max-size 10 \
    --desired-capacity 4 \
    --vpc-zone-identifier "subnet-12345678,subnet-87654321" \
    --health-check-type ELB \
    --health-check-grace-period 300 \
    --target-group-arns "arn:aws:elasticloadbalancing:..."
```

---

## 2. Load Balancing

### 2.1 Elastic Load Balancing (ELB)

**What is ELB?**
- Distributes incoming traffic across multiple EC2 instances
- Improves availability and fault tolerance
- Provides health checking and automatic failover

### 2.2 Load Balancer Types

#### Application Load Balancer (ALB)
**Layer**: Application (Layer 7)
**Protocols**: HTTP, HTTPS
**Features**:
- Content-based routing
- Host-based routing
- Path-based routing
- Query string routing
- SSL/TLS termination
- WebSocket support

**Use Cases**:
- Web applications
- Microservices
- Container-based applications
- HTTP/HTTPS traffic

**Example Routing**:
```
ALB Listener Rules:
├── /api/* → Target Group: API Servers
├── /admin/* → Target Group: Admin Servers
└── /* → Target Group: Web Servers
```

#### Network Load Balancer (NLB)
**Layer**: Network (Layer 4)
**Protocols**: TCP, UDP, TLS
**Features**:
- Ultra-low latency
- High performance
- Static IP addresses
- Preserves source IP
- Zonal isolation

**Use Cases**:
- High-performance applications
- TCP/UDP traffic
- Extreme low latency requirements
- Static IP requirements

#### Classic Load Balancer (CLB)
**Layer**: Application and Network (Layer 4 & 7)
**Legacy**: Older generation, use ALB or NLB for new applications
**Features**:
- Basic load balancing
- Limited routing capabilities

### 2.3 Load Balancer Configuration

**Basic ALB Setup**:
```bash
# Create Target Group
aws elbv2 create-target-group \
    --name web-servers-tg \
    --protocol HTTP \
    --port 80 \
    --vpc-id vpc-12345678 \
    --health-check-path /health

# Register Targets
aws elbv2 register-targets \
    --target-group-arn arn:aws:elasticloadbalancing:... \
    --targets Id=i-1234567890abcdef0 Id=i-0987654321fedcba0

# Create Load Balancer
aws elbv2 create-load-balancer \
    --name web-alb \
    --subnets subnet-12345678 subnet-87654321 \
    --security-groups sg-12345678

# Create Listener
aws elbv2 create-listener \
    --load-balancer-arn arn:aws:elasticloadbalancing:... \
    --protocol HTTP \
    --port 80 \
    --default-actions Type=forward,TargetGroupArn=arn:...
```

### 2.4 Health Checks

**Health Check Configuration**:
- **Protocol**: HTTP, HTTPS, TCP
- **Port**: Port to check
- **Path**: Health check endpoint (for HTTP/HTTPS)
- **Interval**: How often to check (default: 30 seconds)
- **Timeout**: Time to wait for response (default: 5 seconds)
- **Healthy Threshold**: Consecutive successes needed (default: 2)
- **Unhealthy Threshold**: Consecutive failures needed (default: 2)

**Example**:
```json
{
  "HealthCheckProtocol": "HTTP",
  "HealthCheckPath": "/health",
  "HealthCheckIntervalSeconds": 30,
  "HealthCheckTimeoutSeconds": 5,
  "HealthyThresholdCount": 2,
  "UnhealthyThresholdCount": 3
}
```

### 2.5 Load Balancer Best Practices

**High Availability**:
- Deploy across multiple Availability Zones
- Use multiple subnets per AZ
- Enable cross-zone load balancing

**Security**:
- Use security groups to restrict access
- Enable SSL/TLS termination
- Use WAF for web application protection

**Performance**:
- Choose appropriate load balancer type
- Enable connection draining
- Monitor performance metrics

---

## 3. Placement Groups

### 3.1 What are Placement Groups?

**Placement Groups** are logical groupings of instances within a single Availability Zone that influence how instances are placed on underlying hardware.

### 3.2 Placement Group Types

#### Cluster Placement Group
**Use Case**: Low latency, high network throughput
**Characteristics**:
- Instances in same AZ
- Low latency (single-digit millisecond)
- High network throughput (10 Gbps per instance)
- Risk: Single point of failure if AZ fails

**Best For**:
- High-performance computing (HPC)
- Applications requiring low latency
- Big data processing

#### Partition Placement Group
**Use Case**: Large distributed workloads
**Characteristics**:
- Instances spread across partitions
- Up to 7 partitions per AZ
- Instances in same partition share hardware
- Isolated from other partitions

**Best For**:
- Distributed databases (Hadoop, Cassandra)
- Applications requiring partition awareness
- Large-scale distributed systems

#### Spread Placement Group
**Use Case**: Maximum availability
**Characteristics**:
- Instances on distinct hardware
- Maximum 7 instances per AZ
- Isolated from each other
- Best fault tolerance

**Best For**:
- Critical applications
- Small number of instances
- Maximum availability requirements

### 3.3 Placement Group Examples

**Cluster Placement Group**:
```bash
# Create cluster placement group
aws ec2 create-placement-group \
    --group-name hpc-cluster \
    --strategy cluster

# Launch instances in placement group
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type c5.18xlarge \
    --placement GroupName=hpc-cluster \
    --count 4
```

**Partition Placement Group**:
```bash
# Create partition placement group
aws ec2 create-placement-group \
    --group-name cassandra-cluster \
    --strategy partition

# Launch instances in specific partitions
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type r5.2xlarge \
    --placement GroupName=cassandra-cluster,PartitionNumber=1 \
    --count 3
```

---

## 4. Elastic IPs and ENIs

### 4.1 Elastic IP Addresses

**What is an Elastic IP?**
- Static, public IPv4 address
- Associated with your AWS account
- Can be attached/detached from instances
- Retained when instance stops

**Characteristics**:
- **Static**: Doesn't change
- **Reusable**: Can move between instances
- **Cost**: Free when attached to running instance, small charge if unattached
- **Limit**: 5 Elastic IPs per region by default

**Use Cases**:
- Need for static public IP
- DNS records pointing to specific IP
- Failover scenarios

**Example**:
```bash
# Allocate Elastic IP
aws ec2 allocate-address --domain vpc

# Associate with instance
aws ec2 associate-address \
    --instance-id i-1234567890abcdef0 \
    --allocation-id eipalloc-12345678

# Disassociate
aws ec2 disassociate-address --association-id eipassoc-12345678

# Release Elastic IP
aws ec2 release-address --allocation-id eipalloc-12345678
```

### 4.2 Elastic Network Interfaces (ENIs)

**What is an ENI?**
- Virtual network interface that can be attached to EC2 instances
- Can have multiple ENIs per instance
- Can be moved between instances

**Characteristics**:
- **Primary ENI**: Created automatically with instance
- **Secondary ENIs**: Can be created and attached
- **Attributes**: Private IP, public IP, security groups, MAC address
- **Use Cases**: Network management, high availability, security

**Example**:
```bash
# Create ENI
aws ec2 create-network-interface \
    --subnet-id subnet-12345678 \
    --description "Secondary ENI for database"

# Attach to instance
aws ec2 attach-network-interface \
    --network-interface-id eni-12345678 \
    --instance-id i-1234567890abcdef0 \
    --device-index 1
```

---

## 5. IAM Roles for EC2

### 5.1 What are IAM Roles?

**IAM Roles** provide temporary credentials to EC2 instances, eliminating the need to store access keys on instances.

### 5.2 Benefits

- **Security**: No long-term credentials on instances
- **Automatic Rotation**: Credentials automatically rotated
- **Least Privilege**: Grant only necessary permissions
- **Auditability**: All actions logged in CloudTrail

### 5.3 Creating and Attaching IAM Roles

**Step 1: Create IAM Role**:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

**Step 2: Attach Policies**:
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

**Step 3: Create Instance Profile**:
```bash
# Create instance profile
aws iam create-instance-profile --instance-profile-name my-instance-profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
    --instance-profile-name my-instance-profile \
    --role-name my-ec2-role

# Launch instance with role
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t3.micro \
    --iam-instance-profile Name=my-instance-profile
```

### 5.4 Accessing Credentials from Instance

**Instance Metadata Service**:
```bash
# Get temporary credentials
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/role-name

# Response:
{
  "Code": "Success",
  "LastUpdated": "2024-01-01T12:00:00Z",
  "Type": "AWS-HMAC",
  "AccessKeyId": "ASIA...",
  "SecretAccessKey": "...",
  "Token": "...",
  "Expiration": "2024-01-01T18:00:00Z"
}
```

**Using AWS SDK** (automatic):
```python
import boto3

# SDK automatically uses instance role credentials
s3 = boto3.client('s3')
response = s3.list_buckets()
```

---

## 6. Monitoring and Logging

### 6.1 CloudWatch Metrics

**EC2 Metrics** (5-minute default, 1-minute with detailed monitoring):
- **CPUUtilization**: Percentage of CPU utilization
- **NetworkIn/NetworkOut**: Network traffic
- **DiskReadOps/DiskWriteOps**: Disk I/O operations
- **StatusCheckFailed**: Instance or system status check failures

**Enabling Detailed Monitoring**:
```bash
aws ec2 monitor-instances --instance-ids i-1234567890abcdef0
```

### 6.2 CloudWatch Alarms

**Creating Alarms**:
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name high-cpu \
    --alarm-description "Alert when CPU exceeds 80%" \
    --metric-name CPUUtilization \
    --namespace AWS/EC2 \
    --statistic Average \
    --period 300 \
    --threshold 80 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 2 \
    --alarm-actions arn:aws:sns:us-east-1:123456789012:alerts
```

### 6.3 CloudWatch Logs

**Configuring Logs**:
```bash
# Install CloudWatch Logs agent
wget https://s3.amazonaws.com/aws-cloudwatch/downloads/latest/awslogs-agent-setup.py

# Configure agent
sudo python awslogs-agent-setup.py --region us-east-1
```

**Log Groups and Streams**:
- **Log Group**: Container for log streams
- **Log Stream**: Sequence of log events from same source
- **Retention**: Configurable (1 day to never expire)

### 6.4 CloudWatch Dashboard

**Creating Dashboards**:
- Visualize multiple metrics
- Custom widgets and graphs
- Real-time monitoring
- Share with team

**Example Dashboard**:
```
┌─────────────────────────────────────────┐
│ EC2 Instance Dashboard                  │
├─────────────────────────────────────────┤
│ CPU Utilization (4 instances)           │
│ [Graph showing CPU over time]           │
├─────────────────────────────────────────┤
│ Network In/Out                          │
│ [Graph showing network traffic]         │
├─────────────────────────────────────────┤
│ Status Checks                           │
│ [Table showing instance health]         │
└─────────────────────────────────────────┘
```

### 6.5 VPC Flow Logs

**What are Flow Logs?**
- Capture IP traffic going to and from network interfaces
- Help with troubleshooting connectivity issues
- Security monitoring

**Enabling Flow Logs**:
```bash
aws ec2 create-flow-logs \
    --resource-type VPC \
    --resource-ids vpc-12345678 \
    --traffic-type ALL \
    --log-destination-type cloud-watch-logs \
    --log-group-name vpc-flow-logs
```

---

## 7. EC2 Instance Metadata

### 7.1 What is Instance Metadata?

**Instance Metadata** is data about your instance that you can use to configure or manage the running instance.

### 7.2 Accessing Metadata

**Metadata Service Endpoint**: `http://169.254.169.254/latest/meta-data/`

**Common Metadata**:
```bash
# Instance ID
curl http://169.254.169.254/latest/meta-data/instance-id

# Instance Type
curl http://169.254.169.254/latest/meta-data/instance-type

# Availability Zone
curl http://169.254.169.254/latest/meta-data/placement/availability-zone

# Public IP
curl http://169.254.169.254/latest/meta-data/public-ipv4

# Private IP
curl http://169.254.169.254/latest/meta-data/local-ipv4

# IAM Role
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/

# User Data
curl http://169.254.169.254/latest/user-data
```

### 7.3 Instance Metadata Service v2 (IMDSv2)

**Security Enhancement**:
- Session-based authentication
- Protection against SSRF attacks
- Requires PUT request to get token first

**Example**:
```bash
# Get token
TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")

# Use token to access metadata
curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/instance-id
```

---

## 8. Cost Optimization

### 8.1 EC2 Pricing Models

#### On-Demand Instances
- **Pay-as-you-go**: No upfront payment
- **Flexible**: No long-term commitment
- **Use Case**: Short-term, unpredictable workloads
- **Cost**: Highest per-hour rate

#### Reserved Instances (RI)
- **1 or 3 Year Term**: Significant discount (up to 72%)
- **Payment Options**: All upfront, partial upfront, no upfront
- **Use Case**: Predictable, steady-state workloads
- **Cost**: 30-72% savings vs On-Demand

**RI Types**:
- **Standard RIs**: Can be modified (change instance type, AZ)
- **Convertible RIs**: Can be exchanged for different instance types (lower discount)
- **Scheduled RIs**: Reserved for specific time windows

#### Spot Instances
- **Bid-based**: Up to 90% discount
- **Interruptible**: Can be terminated with 2-minute notice
- **Use Case**: Fault-tolerant, flexible workloads
- **Cost**: Lowest (up to 90% savings)

**Spot Instance Best Practices**:
- Use for batch processing, CI/CD, testing
- Implement checkpointing for long-running jobs
- Use Spot Fleet for automatic management
- Diversify across instance types and AZs

#### Savings Plans
- **Flexible Pricing**: 1 or 3 year commitment
- **Compute Savings Plan**: Applies to EC2, Fargate, Lambda
- **EC2 Instance Savings Plan**: Specific to EC2
- **Use Case**: Predictable usage with flexibility
- **Cost**: Up to 72% savings

### 8.2 Cost Optimization Strategies

**Right-Sizing**:
- Match instance type to workload
- Use CloudWatch metrics to identify over-provisioned instances
- Consider burstable instances (T family) for variable workloads

**Scheduling**:
- Stop instances during non-business hours
- Use scheduled scaling for predictable patterns
- Automate start/stop with Lambda and EventBridge

**Reserved Capacity**:
- Purchase RIs for steady-state workloads
- Use Savings Plans for flexible usage
- Analyze usage patterns before purchasing

**Spot Instances**:
- Use for fault-tolerant workloads
- Implement proper interruption handling
- Use Spot Fleet for automatic management

**Storage Optimization**:
- Use appropriate EBS volume types
- Delete unused snapshots
- Use S3 for long-term storage
- Enable EBS volume lifecycle policies

### 8.3 Cost Monitoring

**Cost Explorer**:
- Visualize costs over time
- Identify cost drivers
- Forecast future costs
- Filter by tags, services, etc.

**Cost Allocation Tags**:
- Tag resources for cost tracking
- Track costs by project, department, environment
- Enable cost allocation tags in billing console

**Budgets**:
- Set cost budgets
- Receive alerts when thresholds exceeded
- Take automated actions

---

## 9. High Availability and Disaster Recovery

### 9.1 High Availability Design

**Multi-AZ Deployment**:
```
┌─────────────────────────────────────────┐
│ Region: us-east-1                       │
├─────────────────────────────────────────┤
│                                         │
│  ┌──────────────┐    ┌──────────────┐  │
│  │ Availability │    │ Availability │  │
│  │   Zone 1a    │    │   Zone 1b    │  │
│  │              │    │              │  │
│  │ ┌──────────┐ │    │ ┌──────────┐ │  │
│  │ │ EC2      │ │    │ │ EC2      │ │  │
│  │ │ Instance │ │    │ │ Instance │ │  │
│  │ └──────────┘ │    │ └──────────┘ │  │
│  └──────────────┘    └──────────────┘  │
│         │                   │          │
│         └─────────┬─────────┘          │
│                   │                    │
│            ┌──────▼──────┐             │
│            │ Load       │             │
│            │ Balancer   │             │
│            └────────────┘             │
└─────────────────────────────────────────┘
```

**Key Principles**:
- Deploy across multiple Availability Zones
- Use Auto Scaling for automatic recovery
- Use Load Balancer for traffic distribution
- Implement health checks
- Use Route 53 for DNS failover

### 9.2 Disaster Recovery Strategies

#### Backup and Restore (RTO: Hours, RPO: 24 hours)
- Regular backups to S3
- Restore from backups when needed
- Lowest cost, highest RTO/RPO

#### Pilot Light (RTO: Minutes, RPO: Hours)
- Minimal version of environment always running
- Core services running, scale up on disaster
- Moderate cost, moderate RTO/RPO

#### Warm Standby (RTO: Minutes, RPO: Minutes)
- Scaled-down version always running
- Quick scale-up on disaster
- Higher cost, lower RTO/RPO

#### Multi-Site Active/Active (RTO: Near Zero, RPO: Near Zero)
- Full environment in multiple regions
- Continuous replication
- Highest cost, lowest RTO/RPO

### 9.3 Backup Strategies

**EBS Snapshots**:
- Point-in-time backups
- Incremental (only changed blocks)
- Can be copied across regions
- Automated with Data Lifecycle Manager

**AMI Backups**:
- Complete instance backup
- Includes all attached EBS volumes
- Can be used to launch new instances

**Automated Backups**:
```bash
# Create snapshot
aws ec2 create-snapshot \
    --volume-id vol-1234567890abcdef0 \
    --description "Daily backup"

# Copy snapshot to another region
aws ec2 copy-snapshot \
    --source-region us-east-1 \
    --source-snapshot-id snap-12345678 \
    --region us-west-2
```

---

## 10. Best Practices

### 10.1 Security Best Practices

**Network Security**:
- Use VPCs for network isolation
- Deploy in private subnets when possible
- Use security groups with least privilege
- Implement Network ACLs for additional layer
- Use VPN or Direct Connect for on-premises connectivity

**Instance Security**:
- Use IAM roles instead of access keys
- Enable IMDSv2 for metadata service
- Keep AMIs and software updated
- Use Systems Manager for patching
- Enable CloudTrail for audit logging

**Data Security**:
- Encrypt EBS volumes
- Use encrypted AMIs
- Enable S3 encryption for backups
- Use KMS for key management
- Implement data classification

### 10.2 Performance Best Practices

**Instance Selection**:
- Right-size instances for workload
- Use enhanced networking for high performance
- Consider placement groups for low latency
- Use appropriate EBS volume types

**Application Optimization**:
- Use connection pooling
- Implement caching (ElastiCache)
- Optimize database queries
- Use CDN (CloudFront) for static content

**Monitoring**:
- Enable detailed monitoring
- Set up CloudWatch alarms
- Monitor application metrics
- Use X-Ray for distributed tracing

### 10.3 Operational Best Practices

**Infrastructure as Code**:
- Use CloudFormation, Terraform, or CDK
- Version control infrastructure
- Test changes in non-production
- Use parameterized templates

**Automation**:
- Automate deployments
- Use Auto Scaling
- Implement automated backups
- Use Systems Manager for maintenance

**Tagging**:
- Consistent tagging strategy
- Tag for cost allocation
- Tag for automation
- Tag for compliance

**Documentation**:
- Document architecture
- Document procedures
- Maintain runbooks
- Keep diagrams updated

### 10.4 Cost Optimization Best Practices

**Right-Sizing**:
- Regularly review instance utilization
- Use CloudWatch metrics for analysis
- Consider burstable instances
- Use Spot Instances where appropriate

**Reserved Capacity**:
- Analyze usage patterns
- Purchase RIs for steady workloads
- Use Savings Plans for flexibility
- Consider Convertible RIs for flexibility

**Scheduling**:
- Stop instances when not needed
- Use scheduled scaling
- Automate with Lambda
- Use Spot Instances for dev/test

**Storage**:
- Delete unused snapshots
- Use appropriate volume types
- Enable lifecycle policies
- Archive old data to S3/Glacier

---

## 11. Troubleshooting

### 11.1 Common Issues

#### Instance Won't Start
**Possible Causes**:
- Insufficient capacity in AZ
- Invalid AMI or instance type
- VPC/subnet configuration issues
- IAM permissions

**Solutions**:
- Try different AZ
- Verify AMI and instance type compatibility
- Check VPC and subnet configuration
- Verify IAM permissions

#### Cannot Connect via SSH
**Possible Causes**:
- Security group blocking port 22
- Wrong key pair
- Instance not running
- Network ACL blocking

**Solutions**:
- Check security group rules
- Verify key pair
- Check instance state
- Review Network ACLs
- Use Systems Manager Session Manager

#### High CPU Usage
**Possible Causes**:
- Inefficient application code
- Insufficient instance size
- Malware or unwanted processes
- Resource contention

**Solutions**:
- Profile application
- Upgrade instance type
- Check for malware
- Review CloudWatch metrics
- Consider Auto Scaling

#### Network Connectivity Issues
**Possible Causes**:
- Security group misconfiguration
- Network ACL blocking
- Route table issues
- Internet Gateway not attached

**Solutions**:
- Review security groups
- Check Network ACLs
- Verify route tables
- Check Internet Gateway
- Use VPC Flow Logs

### 11.2 Diagnostic Tools

**CloudWatch Metrics**:
- Monitor instance health
- Identify performance bottlenecks
- Track resource utilization

**VPC Flow Logs**:
- Analyze network traffic
- Troubleshoot connectivity
- Security monitoring

**Systems Manager**:
- Run commands remotely
- View instance information
- Patch management

**CloudTrail**:
- Audit API calls
- Troubleshoot access issues
- Compliance logging

---

## Summary of Part 2

**Key Takeaways**:
1. **Auto Scaling** automatically adjusts capacity based on demand
2. **Load Balancing** distributes traffic across multiple instances for high availability
3. **Placement Groups** optimize instance placement for performance or availability
4. **Elastic IPs** provide static public IP addresses
5. **IAM Roles** provide secure, temporary credentials to instances
6. **Monitoring** with CloudWatch provides visibility into instance health and performance
7. **Cost Optimization** through right-sizing, Reserved Instances, and Spot Instances
8. **High Availability** requires multi-AZ deployment and proper backup strategies
9. **Best Practices** cover security, performance, operations, and cost
10. **Troubleshooting** requires systematic approach using diagnostic tools

**Complete EC2 Knowledge**:
- Part 1 covered fundamentals: instance types, AMIs, storage, networking, security
- Part 2 covered advanced topics: scaling, load balancing, monitoring, optimization

Together, both parts provide comprehensive understanding of EC2 for building scalable, secure, and cost-effective cloud applications.

