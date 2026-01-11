# AWS EC2 In-Depth Interview Guide - Part 1: Fundamentals

## Table of Contents
1. [EC2 Overview](#ec2-overview)
2. [Instance Types & Families](#instance-types--families)
3. [Amazon Machine Images (AMIs)](#amazon-machine-images-amis)
4. [Launching EC2 Instances](#launching-ec2-instances)
5. [Instance Lifecycle](#instance-lifecycle)
6. [Pricing Models](#pricing-models)

---

## EC2 Overview

### What is EC2?

**Amazon Elastic Compute Cloud (EC2)** is a web service that provides resizable compute capacity in the cloud. It allows you to:
- Launch virtual servers (instances) on-demand
- Configure security and networking
- Scale capacity up or down as needed
- Pay only for what you use

### Key Concepts

**Instance**: A virtual server in the AWS cloud
**AMI (Amazon Machine Image)**: Template for your instance (OS, applications, configs)
**Instance Type**: Hardware configuration (CPU, memory, storage, network)
**Security Group**: Virtual firewall controlling inbound/outbound traffic
**Key Pair**: SSH key pair for secure access to Linux instances
**EBS Volume**: Persistent block storage for instances
**VPC**: Virtual network for your instances

### EC2 Benefits

1. **Elasticity**: Scale up/down quickly
2. **Cost-Effective**: Pay only for compute time
3. **Flexibility**: Choose OS, instance type, storage
4. **Reliability**: 99.99% availability SLA
5. **Security**: Multiple layers of security
6. **Integration**: Works with other AWS services

---

## Instance Types & Families

### Instance Type Naming Convention

Format: `[instance-family][generation].[size]`

Examples:
- `t3.micro` - Burstable performance, micro size
- `m5.large` - General purpose, 5th gen, large size
- `c5.xlarge` - Compute optimized, 5th gen, xlarge size

### Instance Families

#### 1. **General Purpose (M, T, Mac)**

**M Family** (M5, M6i, M7i)
- Balanced compute, memory, and networking
- Use cases: Web servers, small databases, development environments
- **Example**: `m5.large` - 2 vCPU, 8 GiB RAM

**T Family** (T3, T4g) - Burstable Performance
- Baseline performance with ability to burst
- CPU Credits system
- Use cases: Web servers, development, small applications
- **Example**: `t3.micro` - 2 vCPU, 1 GiB RAM (burstable)

**Mac Instances** (Mac1, Mac2)
- macOS on AWS
- Use cases: iOS/macOS app development, CI/CD for Apple platforms
- **Example**: `mac1.metal` - 12 vCPU, 32 GiB RAM

#### 2. **Compute Optimized (C)**

**C Family** (C5, C6i, C7i)
- High-performance processors
- Use cases: High-performance computing, scientific modeling, gaming servers
- **Example**: `c5.xlarge` - 4 vCPU, 8 GiB RAM

#### 3. **Memory Optimized (R, X, High Memory)**

**R Family** (R5, R6i, R7i)
- High memory-to-CPU ratio
- Use cases: In-memory databases, real-time big data analytics
- **Example**: `r5.2xlarge` - 8 vCPU, 64 GiB RAM

**X Family** (X1, X1e)
- Very high memory
- Use cases: SAP HANA, large in-memory databases
- **Example**: `x1.32xlarge` - 128 vCPU, 1,952 GiB RAM

**High Memory** (u-6tb1, u-9tb1, u-12tb1)
- Ultra-high memory instances
- Use cases: Very large in-memory databases
- **Example**: `u-6tb1.metal` - 448 vCPU, 6,144 GiB RAM

#### 4. **Storage Optimized (I, D, H1)**

**I Family** (I3, I4i)
- High IOPS SSD storage
- Use cases: NoSQL databases, data warehousing, OLTP
- **Example**: `i3.xlarge` - 4 vCPU, 30.5 GiB RAM, 1x 950 GB NVMe SSD

**D Family** (D2, D3)
- Dense storage instances
- Use cases: Hadoop, data warehousing, log processing
- **Example**: `d2.xlarge` - 4 vCPU, 30.5 GiB RAM, 3x 2,000 GB HDD

**H1 Family**
- High disk throughput
- Use cases: MapReduce, distributed file systems
- **Example**: `h1.2xlarge` - 8 vCPU, 32 GiB RAM, 1x 2,000 GB HDD

#### 5. **Accelerated Computing (P, G, Inf, Trn1)**

**P Family** (P3, P4)
- GPU instances for parallel processing
- Use cases: Machine learning, high-performance computing
- **Example**: `p3.2xlarge` - 8 vCPU, 61 GiB RAM, 1x NVIDIA V100 GPU

**G Family** (G4, G5)
- GPU instances for graphics-intensive applications
- Use cases: Video encoding, 3D rendering, game streaming
- **Example**: `g4dn.xlarge` - 4 vCPU, 16 GiB RAM, 1x NVIDIA T4 GPU

**Inf1**
- AWS Inferentia chips for machine learning inference
- Use cases: ML inference at scale
- **Example**: `inf1.xlarge` - 4 vCPU, 8 GiB RAM, 1x AWS Inferentia

**Trn1**
- Trainium chips for machine learning training
- Use cases: ML model training
- **Example**: `trn1.2xlarge` - 8 vCPU, 32 GiB RAM, 1x AWS Trainium

### Instance Size Comparison

| Size | vCPU | RAM (GiB) | Network Performance |
|------|------|-----------|-------------------|
| nano | 2 | 0.5 | Up to 5 Gbps |
| micro | 2 | 1 | Up to 5 Gbps |
| small | 2 | 2 | Up to 10 Gbps |
| medium | 2 | 4 | Up to 10 Gbps |
| large | 2 | 8 | Up to 10 Gbps |
| xlarge | 4 | 16 | Up to 10 Gbps |
| 2xlarge | 8 | 32 | Up to 10 Gbps |
| 4xlarge | 16 | 64 | Up to 10 Gbps |
| 8xlarge | 32 | 128 | 10 Gbps |
| 12xlarge | 48 | 192 | 10 Gbps |
| 16xlarge | 64 | 256 | 20 Gbps |
| 24xlarge | 96 | 384 | 25 Gbps |
| 32xlarge | 128 | 512 | 25 Gbps |
| metal | Varies | Varies | 25-100 Gbps |

### Choosing the Right Instance Type

**Considerations:**
1. **CPU Requirements**: Compute-intensive vs. general purpose
2. **Memory Requirements**: RAM needs for applications
3. **Storage Requirements**: IOPS, throughput, capacity
4. **Network Requirements**: Bandwidth needs
5. **Cost**: Balance performance with budget
6. **Workload Type**: CPU-bound, memory-bound, I/O-bound

**Decision Tree:**
```
Is it CPU-intensive? → C family
Is it memory-intensive? → R/X family
Is it storage-intensive? → I/D family
Is it general purpose? → M family
Is it low-traffic/bursty? → T family
Does it need GPU? → P/G family
Does it need ML inference? → Inf1
Does it need ML training? → Trn1
```

---

## Amazon Machine Images (AMIs)

### What is an AMI?

An **AMI** is a template that contains:
- Operating system (Linux, Windows, macOS)
- Application server and applications
- Configuration settings
- Block device mappings (volumes)

### AMI Types

#### 1. **Public AMIs**
- Provided by AWS, community, or marketplace
- Free or paid (marketplace)
- Examples: Amazon Linux 2, Ubuntu, Windows Server

#### 2. **Private AMIs**
- Created by you or shared with your account
- Not publicly accessible
- Use cases: Custom configurations, compliance requirements

#### 3. **AWS Marketplace AMIs**
- Pre-configured software on AMIs
- May have licensing costs
- Examples: WordPress, LAMP stack, security tools

### AMI Components

1. **Root Volume**: Contains OS and boot files
2. **Block Device Mapping**: Defines volumes attached to instance
3. **Launch Permissions**: Who can use the AMI
4. **Product Codes**: For marketplace AMIs
5. **Kernel ID**: For paravirtual AMIs (legacy)

### AMI Lifecycle

```
Create Snapshot → Create AMI → Launch Instance → Modify → Create New AMI
```

### Hands-On: Working with AMIs

#### List Available AMIs

```bash
# List Amazon Linux 2 AMIs
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*" \
            "Name=architecture,Values=x86_64" \
            "Name=root-device-type,Values=ebs" \
  --query 'Images[*].[ImageId,Name,CreationDate]' \
  --output table \
  --region us-east-1

# List Ubuntu AMIs
aws ec2 describe-images \
  --owners 099720109477 \
  --filters "Name=name,Values=ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*" \
            "Name=architecture,Values=x86_64" \
  --query 'Images[*].[ImageId,Name,CreationDate]' \
  --output table \
  --region us-east-1
```

#### Create Custom AMI from Running Instance

```bash
# Step 1: Stop the instance (optional, but recommended)
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Step 2: Create AMI from instance
aws ec2 create-image \
  --instance-id i-1234567890abcdef0 \
  --name "my-custom-ami-$(date +%Y%m%d)" \
  --description "Custom AMI with application installed" \
  --no-reboot

# Output: ImageId (e.g., ami-0abcdef1234567890)
```

#### Copy AMI to Another Region

```bash
# Copy AMI from us-east-1 to us-west-2
aws ec2 copy-image \
  --source-region us-east-1 \
  --source-image-id ami-0abcdef1234567890 \
  --name "my-custom-ami-copy" \
  --description "Copied AMI" \
  --region us-west-2
```

#### Deregister AMI

```bash
# First, get snapshot ID
aws ec2 describe-images --image-ids ami-0abcdef1234567890 \
  --query 'Images[0].BlockDeviceMappings[0].Ebs.SnapshotId'

# Deregister AMI
aws ec2 deregister-image --image-id ami-0abcdef1234567890

# Delete snapshot (optional, to save costs)
aws ec2 delete-snapshot --snapshot-id snap-0abcdef1234567890
```

---

## Launching EC2 Instances

### Launch Methods

1. **AWS Console**: Web-based interface
2. **AWS CLI**: Command-line interface
3. **AWS SDK**: Programmatic access
4. **CloudFormation**: Infrastructure as code
5. **Terraform**: Infrastructure as code
6. **Launch Templates**: Reusable launch configurations

### Launch Process

1. **Choose AMI**: Select OS and software
2. **Choose Instance Type**: Select hardware configuration
3. **Configure Instance**: Network, IAM role, user data
4. **Add Storage**: EBS volumes, instance store
5. **Configure Security Group**: Firewall rules
6. **Review and Launch**: Key pair selection

### Hands-On: Launching Instances

#### Basic Instance Launch

```bash
# Launch a t3.micro instance with Amazon Linux 2
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --count 1 \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]'
```

#### Launch with User Data (Bootstrap Script)

```bash
# Create user-data script
cat > user-data.sh << 'EOF'
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2</h1>" > /var/www/html/index.html
EOF

# Launch instance with user data
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --user-data file://user-data.sh \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]'
```

#### Launch with IAM Role

```bash
# Create IAM role for EC2 (requires IAM policy)
aws iam create-role \
  --role-name EC2-S3-Access-Role \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

# Attach policy to role
aws iam attach-role-policy \
  --role-name EC2-S3-Access-Role \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

# Create instance profile
aws iam create-instance-profile \
  --instance-profile-name EC2-S3-Access-Profile

# Add role to instance profile
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-S3-Access-Profile \
  --role-name EC2-S3-Access-Role

# Launch instance with IAM role
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --iam-instance-profile Name=EC2-S3-Access-Profile \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]'
```

#### Launch with Multiple EBS Volumes

```bash
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0 \
  --block-device-mappings '[
    {
      "DeviceName": "/dev/xvda",
      "Ebs": {
        "VolumeSize": 20,
        "VolumeType": "gp3",
        "DeleteOnTermination": true
      }
    },
    {
      "DeviceName": "/dev/xvdf",
      "Ebs": {
        "VolumeSize": 100,
        "VolumeType": "gp3",
        "DeleteOnTermination": false
      }
    }
  ]' \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]'
```

#### Launch Spot Instance

```bash
# Request spot instance
aws ec2 request-spot-instances \
  --spot-price "0.05" \
  --instance-count 1 \
  --type "one-time" \
  --launch-specification '{
    "ImageId": "ami-0c55b159cbfafe1f0",
    "InstanceType": "t3.micro",
    "KeyName": "my-key-pair",
    "SecurityGroupIds": ["sg-0123456789abcdef0"],
    "SubnetId": "subnet-0123456789abcdef0"
  }'
```

---

## Instance Lifecycle

### Instance States

1. **pending**: Instance is launching
2. **running**: Instance is running and ready
3. **stopping**: Instance is being stopped
4. **stopped**: Instance is stopped (not terminated)
5. **shutting-down**: Instance is being terminated
6. **terminated**: Instance is terminated (cannot be started)

### State Transitions

```
pending → running → stopping → stopped → (can restart) → running
running → shutting-down → terminated (cannot restart)
```

### Hands-On: Managing Instance Lifecycle

#### Check Instance Status

```bash
# Describe instance status
aws ec2 describe-instance-status \
  --instance-ids i-1234567890abcdef0

# Get detailed instance information
aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress,PrivateIpAddress]' \
  --output table
```

#### Start Instance

```bash
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Wait for instance to be running
aws ec2 wait instance-running --instance-ids i-1234567890abcdef0
```

#### Stop Instance

```bash
# Stop instance (graceful shutdown)
aws ec2 stop-instances --instance-ids i-1234567890abcdef0

# Force stop (hard shutdown)
aws ec2 stop-instances --instance-ids i-1234567890abcdef0 --force

# Wait for instance to be stopped
aws ec2 wait instance-stopped --instance-ids i-1234567890abcdef0
```

#### Reboot Instance

```bash
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0
```

#### Terminate Instance

```bash
# Terminate instance (cannot be undone)
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0

# Wait for termination
aws ec2 wait instance-terminated --instance-ids i-1234567890abcdef0
```

#### Modify Instance Attributes

```bash
# Change instance type (must be stopped)
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 wait instance-stopped --instance-ids i-1234567890abcdef0

aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --instance-type Value=t3.small

aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Enable termination protection
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --disable-api-termination

# Change security groups (requires VPC)
aws ec2 modify-instance-attribute \
  --instance-id i-1234567890abcdef0 \
  --groups sg-0123456789abcdef0 sg-0987654321fedcba0
```

---

## Pricing Models

### 1. **On-Demand Instances**

**Characteristics:**
- Pay per second (minimum 60 seconds)
- No upfront costs
- No long-term commitments
- Highest cost per hour

**Use Cases:**
- Short-term, unpredictable workloads
- Testing and development
- Applications that can't be interrupted

**Pricing Example:**
- `t3.micro` in us-east-1: ~$0.0104/hour
- `m5.large` in us-east-1: ~$0.096/hour

#### Calculate Monthly Cost

```bash
# Calculate monthly cost for t3.micro
# $0.0104/hour * 24 hours * 30 days = $7.49/month
```

### 2. **Reserved Instances (RIs)**

**Characteristics:**
- Significant discount (up to 72% off On-Demand)
- 1 or 3-year term commitment
- Payment options: All Upfront, Partial Upfront, No Upfront
- Can be sold in Reserved Instance Marketplace

**Types:**
- **Standard RIs**: Can modify instance attributes
- **Convertible RIs**: Can change instance family, OS, tenancy (up to 66% discount)
- **Scheduled RIs**: Launch within time windows

**Use Cases:**
- Predictable, steady-state workloads
- Applications requiring reserved capacity
- Cost optimization for long-running instances

**Pricing Example:**
- `m5.large` Standard RI (1-year, All Upfront): ~$0.06/hour (37% discount)
- `m5.large` Standard RI (3-year, All Upfront): ~$0.04/hour (58% discount)

### 3. **Savings Plans**

**Characteristics:**
- Flexible pricing model
- Commit to $/hour usage (1 or 3 years)
- Applies to EC2, Lambda, Fargate
- Can change instance family, size, region, OS

**Types:**
- **Compute Savings Plans**: Most flexible (up to 66% discount)
- **EC2 Instance Savings Plans**: EC2-specific (up to 72% discount)

**Use Cases:**
- Flexible workloads with changing requirements
- Mix of EC2, Lambda, Fargate usage
- Want flexibility to change instance types

### 4. **Spot Instances**

**Characteristics:**
- Up to 90% discount off On-Demand
- Can be interrupted by AWS with 2-minute notice
- Best for fault-tolerant, flexible workloads
- Spot Fleet for automatic management

**Use Cases:**
- Batch processing
- Data analysis
- CI/CD workloads
- Web services with flexible start/end times

**Pricing Example:**
- `m5.large` Spot: ~$0.03/hour (70% discount, varies by demand)

#### Request Spot Instances

```bash
# Request spot instance with maximum price
aws ec2 request-spot-instances \
  --spot-price "0.05" \
  --instance-count 1 \
  --type "persistent" \
  --valid-from "2024-01-01T00:00:00Z" \
  --valid-until "2024-12-31T23:59:59Z" \
  --launch-specification '{
    "ImageId": "ami-0c55b159cbfafe1f0",
    "InstanceType": "t3.micro",
    "KeyName": "my-key-pair",
    "SecurityGroupIds": ["sg-0123456789abcdef0"],
    "SubnetId": "subnet-0123456789abcdef0"
  }'
```

### 5. **Dedicated Hosts**

**Characteristics:**
- Physical server dedicated to your use
- Full control over instance placement
- Can use existing server-bound software licenses
- Most expensive option

**Use Cases:**
- Compliance requirements
- Server-bound software licenses
- Regulatory requirements

### 6. **Dedicated Instances**

**Characteristics:**
- Instances on hardware dedicated to you
- May share hardware with other instances in your account
- No control over instance placement
- Additional per-hour charge

### Cost Optimization Strategies

1. **Right-Sizing**: Match instance type to workload
2. **Reserved Instances**: For steady-state workloads
3. **Savings Plans**: For flexible workloads
4. **Spot Instances**: For fault-tolerant workloads
5. **Auto Scaling**: Scale down during low usage
6. **Terminate Unused Instances**: Clean up development instances
7. **Use Spot for Batch Jobs**: Significant cost savings

#### Calculate Cost Savings

```bash
# Example: Compare On-Demand vs Reserved Instance
# On-Demand: m5.large = $0.096/hour = $70.08/month
# Reserved (1-year): m5.large = $0.06/hour = $43.80/month
# Savings: $26.28/month (37% discount)

# Annual savings: $315.36
```

---

## Interview Questions & Answers

### Q1: What is the difference between stopping and terminating an instance?

**Answer:**
- **Stopping**: Instance is shut down but not deleted. EBS volumes persist. Can be restarted. Charged for EBS storage only.
- **Terminating**: Instance is permanently deleted. Cannot be restarted. EBS volumes are deleted if `DeleteOnTermination=true`.

### Q2: What happens to data when an instance is terminated?

**Answer:**
- **Root EBS Volume**: Deleted if `DeleteOnTermination=true` (default for most AMIs)
- **Additional EBS Volumes**: Persist by default unless explicitly set to delete
- **Instance Store**: Data is lost (ephemeral storage)
- **Elastic IP**: Released if not disassociated
- **Public IP**: Released

### Q3: How do you choose between different instance families?

**Answer:**
- **M Family**: General purpose, balanced resources
- **C Family**: CPU-intensive workloads (HPC, gaming)
- **R Family**: Memory-intensive (databases, analytics)
- **I Family**: High IOPS storage (NoSQL, data warehousing)
- **T Family**: Burstable workloads (web servers, dev/test)
- **P/G Family**: GPU workloads (ML, graphics)

### Q4: What is the difference between On-Demand and Spot Instances?

**Answer:**
- **On-Demand**: Fixed price, always available, no interruption
- **Spot**: Variable price (up to 90% discount), can be interrupted with 2-minute notice, best for fault-tolerant workloads

### Q5: How do AMIs work across regions?

**Answer:**
- AMIs are region-specific. Each region has its own AMI ID.
- To use an AMI in another region, you must:
  1. Copy the AMI to the target region
  2. Use the new AMI ID in that region
- AMI copying creates a new AMI ID in the target region.

---

## Summary

**Key Takeaways:**
1. EC2 provides resizable compute capacity in the cloud
2. Instance types are optimized for different workloads (CPU, memory, storage, GPU)
3. AMIs are templates containing OS, applications, and configurations
4. Instances have a lifecycle: pending → running → stopping → stopped → terminated
5. Multiple pricing models: On-Demand, Reserved, Savings Plans, Spot, Dedicated
6. Cost optimization is crucial: right-sizing, Reserved Instances, Spot for fault-tolerant workloads

**Next Part**: Part 2 will cover Storage (EBS, Instance Store), Networking, Security Groups, and Key Pairs.

---

**Part 1 Complete** - Ready for Part 2 on prompt

