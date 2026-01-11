# AWS EC2 (Elastic Compute Cloud) - In-Depth Guide

## Part 1: Fundamentals, Instance Types, and Core Concepts

---

## Table of Contents

1. [Introduction to EC2](#1-introduction-to-ec2)
2. [EC2 Instance Types](#2-ec2-instance-types)
3. [Amazon Machine Images (AMIs)](#3-amazon-machine-images-amis)
4. [Launching EC2 Instances](#4-launching-ec2-instances)
5. [EC2 Instance Lifecycle](#5-ec2-instance-lifecycle)
6. [Storage Options](#6-storage-options)
7. [Networking Fundamentals](#7-networking-fundamentals)
8. [Security Groups](#8-security-groups)
9. [Key Pairs](#9-key-pairs)

---

## 1. Introduction to EC2

### 1.1 What is EC2?

**Amazon Elastic Compute Cloud (EC2)** is a web service that provides resizable compute capacity in the cloud. It allows you to launch virtual servers (instances) on-demand, configure security and networking, and manage storage.

### 1.2 Key Characteristics

- **Elastic**: Scale capacity up or down automatically
- **Virtual Servers**: Run on AWS's virtualization infrastructure
- **On-Demand**: Launch instances in minutes
- **Pay-as-you-go**: Pay only for what you use
- **Flexible**: Choose instance types, operating systems, and configurations
- **Secure**: Built-in security features and network isolation

### 1.3 Use Cases

- **Web Applications**: Hosting web servers and application backends
- **Development/Testing**: Creating isolated environments
- **Data Processing**: Running batch jobs and analytics workloads
- **High-Performance Computing**: Scientific computing and simulations
- **Machine Learning**: Training and inference workloads
- **Enterprise Applications**: Running business-critical applications

### 1.4 EC2 Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AWS Region                            │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │           Availability Zone (AZ-1)              │   │
│  │  ┌──────────────┐  ┌──────────────┐             │   │
│  │  │ EC2 Instance │  │ EC2 Instance │             │   │
│  │  │  (t3.medium) │  │  (m5.large)  │             │   │
│  │  └──────────────┘  └──────────────┘             │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │
│  │           Availability Zone (AZ-2)              │   │
│  │  ┌──────────────┐  ┌──────────────┐             │   │
│  │  │ EC2 Instance │  │ EC2 Instance │             │   │
│  │  │  (c5.xlarge) │  │  (r5.2xlarge)│             │   │
│  │  └──────────────┘  └──────────────┘             │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 2. EC2 Instance Types

### 2.1 Instance Type Naming Convention

EC2 instance types follow a naming pattern: `family.size`

**Format**: `[instance-family].[instance-size]`

**Example**: `t3.medium`
- `t3` = Instance family (general purpose, burstable)
- `medium` = Instance size (CPU, memory, network capacity)

### 2.2 Instance Families

#### General Purpose (T, M, Mac)

**T Family (Burstable Performance)**
- **Use Case**: Variable workloads, web servers, development
- **Characteristics**: Baseline performance with ability to burst
- **Examples**: `t3.micro`, `t3.small`, `t3.medium`, `t3.large`, `t3.xlarge`
- **Burst Credits**: Earn credits when idle, spend when busy
- **Best For**: Applications with variable CPU usage

**M Family (General Purpose)**
- **Use Case**: Balanced compute, memory, and networking
- **Characteristics**: Balanced resources for most workloads
- **Examples**: `m5.large`, `m5.xlarge`, `m5.2xlarge`, `m5.4xlarge`
- **Best For**: Application servers, small to medium databases, web servers

**Mac Instances**
- **Use Case**: macOS workloads, iOS app development
- **Characteristics**: Dedicated Mac mini hardware
- **Examples**: `mac1.metal`
- **Best For**: Building and testing macOS and iOS applications

#### Compute Optimized (C)

**C Family**
- **Use Case**: Compute-intensive workloads
- **Characteristics**: High CPU-to-memory ratio
- **Examples**: `c5.large`, `c5.xlarge`, `c5.2xlarge`, `c5n.xlarge` (network optimized)
- **Best For**: Batch processing, scientific modeling, gaming servers, ad serving

#### Memory Optimized (R, X, High Memory)

**R Family**
- **Use Case**: Memory-intensive applications
- **Characteristics**: High memory-to-CPU ratio
- **Examples**: `r5.large`, `r5.xlarge`, `r5.2xlarge`, `r5.4xlarge`
- **Best For**: High-performance databases, in-memory caches, real-time big data analytics

**X Family (X1, X1e)**
- **Use Case**: Very large in-memory databases
- **Characteristics**: Extremely high memory
- **Examples**: `x1.32xlarge` (2TB RAM), `x1e.32xlarge` (4TB RAM)
- **Best For**: SAP HANA, large-scale in-memory databases

**High Memory Instances**
- **Use Case**: Largest memory requirements
- **Characteristics**: Up to 24TB RAM
- **Examples**: `u-6tb1.metal`, `u-9tb1.metal`, `u-12tb1.metal`
- **Best For**: Largest in-memory databases, SAP HANA scale-up

#### Storage Optimized (I, D, H1)

**I Family**
- **Use Case**: High I/O databases
- **Characteristics**: High IOPS, NVMe SSD storage
- **Examples**: `i3.large`, `i3.xlarge`, `i3.2xlarge`, `i3en.3xlarge`
- **Best For**: NoSQL databases (Cassandra, MongoDB), data warehousing, OLTP

**D Family**
- **Use Case**: Dense storage instances
- **Characteristics**: High storage density
- **Examples**: `d2.xlarge`, `d2.2xlarge`, `d2.4xlarge`
- **Best For**: Hadoop/MapReduce, data warehousing, log processing

**H1 Family**
- **Use Case**: Big data workloads
- **Characteristics**: High disk throughput
- **Examples**: `h1.2xlarge`, `h1.4xlarge`, `h1.8xlarge`
- **Best For**: MapReduce, distributed file systems, data processing

#### Accelerated Computing (P, G, Inf, F1)

**P Family (GPU)**
- **Use Case**: Machine learning, high-performance computing
- **Characteristics**: NVIDIA GPUs
- **Examples**: `p3.2xlarge`, `p3.8xlarge`, `p3.16xlarge`, `p4d.24xlarge`
- **Best For**: Deep learning, scientific computing, rendering

**G Family (GPU)**
- **Use Case**: Graphics-intensive applications
- **Characteristics**: NVIDIA GPUs for graphics
- **Examples**: `g4dn.xlarge`, `g4dn.2xlarge`
- **Best For**: Video encoding, 3D rendering, machine learning inference

**Inf Family (Inferentia)**
- **Use Case**: Machine learning inference
- **Characteristics**: AWS Inferentia chips
- **Examples**: `inf1.xlarge`, `inf1.2xlarge`
- **Best For**: Machine learning inference at scale

**F1 Family (FPGA)**
- **Use Case**: Custom hardware acceleration
- **Characteristics**: Field Programmable Gate Arrays
- **Examples**: `f1.2xlarge`, `f1.4xlarge`, `f1.16xlarge`
- **Best For**: Custom hardware acceleration, genomics, financial analytics

### 2.3 Instance Sizes

| Size | vCPUs | Memory (GB) | Network Performance | Use Case |
|------|-------|-------------|---------------------|----------|
| **nano** | 2 | 0.5 | Up to 5 Gbps | Development, testing |
| **micro** | 2 | 1 | Up to 5 Gbps | Low-traffic websites |
| **small** | 2 | 2 | Up to 10 Gbps | Small applications |
| **medium** | 2 | 4 | Up to 10 Gbps | Medium applications |
| **large** | 2 | 8 | Up to 10 Gbps | Production workloads |
| **xlarge** | 4 | 16 | Up to 10 Gbps | High-performance apps |
| **2xlarge** | 8 | 32 | Up to 10 Gbps | Large applications |
| **4xlarge** | 16 | 64 | Up to 10 Gbps | Very large workloads |
| **8xlarge** | 32 | 128 | 10 Gbps | Enterprise applications |
| **12xlarge** | 48 | 192 | 10 Gbps | Large-scale processing |
| **16xlarge** | 64 | 256 | 25 Gbps | Massive workloads |
| **24xlarge** | 96 | 384 | 25 Gbps | Extreme performance |
| **metal** | All | All | 100 Gbps | Bare metal performance |

### 2.4 Choosing the Right Instance Type

**Decision Factors**:
1. **CPU Requirements**: Compute-intensive → C family
2. **Memory Requirements**: Memory-intensive → R family
3. **Storage I/O**: High I/O → I family
4. **Network Performance**: High throughput → Enhanced networking
5. **Cost**: Budget constraints → T family (burstable)
6. **Workload Type**: General purpose → M family

**Example Decision Tree**:
```
Need GPU?
├─ Yes → P or G family
└─ No → Continue

High memory requirements?
├─ Yes → R or X family
└─ No → Continue

High CPU requirements?
├─ Yes → C family
└─ No → Continue

High storage I/O?
├─ Yes → I or D family
└─ No → M or T family (general purpose)
```

---

## 3. Amazon Machine Images (AMIs)

### 3.1 What is an AMI?

An **Amazon Machine Image (AMI)** is a template that contains the software configuration (operating system, application server, and applications) required to launch an EC2 instance.

### 3.2 AMI Components

1. **Root Volume Template**: Operating system and applications
2. **Launch Permissions**: Who can use the AMI
3. **Block Device Mapping**: Volumes to attach to the instance
4. **Kernel ID**: Kernel to use (for paravirtual AMIs)

### 3.3 Types of AMIs

#### Public AMIs
- Provided by AWS, community, or third parties
- Free or paid
- Examples: Amazon Linux 2, Ubuntu, Windows Server

#### Private AMIs
- Created by you or your organization
- Only accessible to your AWS account
- Custom configurations

#### Marketplace AMIs
- Pre-configured software stacks
- May have licensing fees
- Examples: WordPress, LAMP stack, security tools

### 3.4 AMI Categories

**By Operating System**:
- **Linux**: Amazon Linux, Ubuntu, Red Hat, SUSE, Debian
- **Windows**: Windows Server 2019, 2022, Windows 10
- **macOS**: macOS Big Sur, Monterey

**By Virtualization Type**:
- **HVM (Hardware Virtual Machine)**: Full virtualization, better performance
- **PV (Paravirtual)**: Older type, limited support

**By Root Device Type**:
- **EBS-backed**: Root volume is EBS, can be stopped/started
- **Instance Store-backed**: Root volume is ephemeral, cannot be stopped

### 3.5 Creating Custom AMIs

**Use Cases**:
- Pre-configured application stacks
- Security-hardened images
- Compliance requirements
- Faster instance launches

**Process**:
1. Launch instance from base AMI
2. Install and configure software
3. Create AMI from instance
4. Use AMI to launch new instances

**Example**:
```bash
# Launch instance
aws ec2 run-instances --image-id ami-0abcdef1234567890 --instance-type t3.micro

# Configure instance (install software, etc.)

# Create AMI
aws ec2 create-image --instance-id i-1234567890abcdef0 --name "my-custom-ami"
```

---

## 4. Launching EC2 Instances

### 4.1 Launch Methods

#### AWS Console
- Web-based graphical interface
- Step-by-step wizard
- Good for beginners

#### AWS CLI
- Command-line interface
- Scriptable and automatable
- Good for automation

#### AWS SDK
- Programmatic access
- Integration with applications
- Good for application integration

#### Infrastructure as Code
- CloudFormation, Terraform, CDK
- Version-controlled infrastructure
- Good for production deployments

### 4.2 Launch Configuration Parameters

**Required Parameters**:
- **AMI ID**: Which image to use
- **Instance Type**: Size and capabilities
- **Key Pair**: For SSH access (Linux) or RDP (Windows)
- **Security Group**: Firewall rules
- **Subnet**: Network location (optional, uses default VPC if not specified)

**Optional Parameters**:
- **User Data**: Scripts to run at launch
- **IAM Role**: Permissions for the instance
- **Storage**: EBS volumes to attach
- **Tags**: Metadata for organization
- **Tenancy**: Shared, dedicated, or dedicated host

### 4.3 Launch Example (AWS CLI)

```bash
# Launch EC2 instance
aws ec2 run-instances \
    --image-id ami-0c55b159cbfafe1f0 \
    --instance-type t3.micro \
    --key-name my-key-pair \
    --security-group-ids sg-12345678 \
    --subnet-id subnet-12345678 \
    --user-data file://user-data.sh \
    --iam-instance-profile Name=MyInstanceProfile \
    --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=WebServer}]' \
    --count 1
```

### 4.4 User Data Scripts

User data scripts run when the instance first launches. They can install software, configure services, or perform setup tasks.

**Linux Example**:
```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from EC2</h1>" > /var/www/html/index.html
```

**Windows Example (PowerShell)**:
```powershell
<powershell>
Install-WindowsFeature -name Web-Server -IncludeManagementTools
Set-Content -Path "C:\inetpub\wwwroot\index.html" -Value "<h1>Hello from EC2</h1>"
</powershell>
```

---

## 5. EC2 Instance Lifecycle

### 5.1 Instance States

```
┌─────────┐
│ pending │ ← Instance is launching
└────┬────┘
     │
     ▼
┌─────────┐
│ running │ ← Instance is running and ready
└────┬────┘
     │
     ├──► ┌──────────┐
     │    │ stopping │ ← Instance is shutting down
     │    └────┬─────┘
     │         │
     │         ▼
     │    ┌──────────┐
     │    │ stopped  │ ← Instance is stopped (EBS-backed only)
     │    └────┬─────┘
     │         │
     │         ▼
     │    ┌──────────┐
     │    │ starting │ ← Instance is starting
     │    └────┬─────┘
     │         │
     │         └──► ┌─────────┐
     │              │ running │
     │              └─────────┘
     │
     ├──► ┌──────────┐
     │    │shutting-│ ← Instance is terminating
     │    │  down   │
     │    └────┬────┘
     │         │
     │         ▼
     │    ┌──────────┐
     │    │terminated│ ← Instance is terminated
     │    └──────────┘
     │
     └──► ┌──────────┐
          │ rebooting│ ← Instance is rebooting
          └────┬─────┘
               │
               └──► ┌─────────┐
                    │ running │
                    └─────────┘
```

### 5.2 State Transitions

**Pending → Running**:
- Instance is being launched
- Billing starts when state becomes "running"
- Typically takes 1-2 minutes

**Running → Stopping → Stopped**:
- Only for EBS-backed instances
- Instance is shut down but not terminated
- Data on EBS volumes is preserved
- Can be started again later
- Billing stops (except for EBS storage)

**Stopped → Starting → Running**:
- Instance is being started
- New instance ID may be assigned
- Billing resumes

**Running → Rebooting → Running**:
- Instance is restarted
- Same instance ID
- Billing continues

**Running → Shutting-down → Terminated**:
- Instance is being terminated
- Cannot be stopped or started
- EBS volumes can be preserved (if configured)
- Billing stops

### 5.3 Instance Lifecycle Management

**Start Instance**:
```bash
aws ec2 start-instances --instance-ids i-1234567890abcdef0
```

**Stop Instance**:
```bash
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

**Reboot Instance**:
```bash
aws ec2 reboot-instances --instance-ids i-1234567890abcdef0
```

**Terminate Instance**:
```bash
aws ec2 terminate-instances --instance-ids i-1234567890abcdef0
```

---

## 6. Storage Options

### 6.1 EBS (Elastic Block Store)

**Characteristics**:
- Persistent block storage
- Attached to EC2 instances
- Survives instance termination (if configured)
- Can be detached and reattached
- Multiple volume types available

**Volume Types**:

| Type | Use Case | IOPS | Throughput | Cost |
|------|----------|------|-------------|------|
| **gp3** | General purpose (SSD) | 3,000-16,000 | 125-1,000 MB/s | Low |
| **gp2** | General purpose (SSD) | 3-16,000 | 128-250 MB/s | Low |
| **io1/io2** | High IOPS (SSD) | 100-256,000 | Up to 4,000 MB/s | High |
| **st1** | Throughput optimized (HDD) | 500 | 500 MB/s | Very Low |
| **sc1** | Cold HDD | 250 | 250 MB/s | Lowest |

**EBS Volume Features**:
- **Snapshots**: Point-in-time backups
- **Encryption**: Data encrypted at rest
- **Multi-Attach**: Attach to multiple instances (io1/io2)
- **Resize**: Increase size without downtime
- **Change Type**: Migrate between volume types

### 6.2 Instance Store

**Characteristics**:
- Ephemeral storage
- Physically attached to host
- Very high I/O performance
- Data lost on stop/terminate
- Free (included with instance)

**Use Cases**:
- Temporary data
- Cache, buffers
- Scratch space
- Data that can be regenerated

### 6.3 EFS (Elastic File System)

**Characteristics**:
- Managed NFS file system
- Shared across multiple instances
- Scales automatically
- Pay for storage used
- High availability and durability

**Use Cases**:
- Shared application files
- Content management
- Web serving
- Big data analytics

### 6.4 Storage Best Practices

**Root Volume**:
- Use EBS-backed for production
- Enable encryption
- Keep size minimal (8-20 GB typically)
- Use snapshots for backup

**Data Volumes**:
- Separate volumes for data and logs
- Use appropriate volume type for workload
- Enable encryption for sensitive data
- Regular snapshots for backup

**Instance Store**:
- Use for temporary/cache data only
- Don't store critical data
- Consider for high I/O workloads

---

## 7. Networking Fundamentals

### 7.1 VPC (Virtual Private Cloud)

**What is VPC?**
- Isolated virtual network in AWS
- Complete control over network configuration
- Can span multiple Availability Zones
- Default VPC provided in each region

**VPC Components**:
- **Subnets**: Segments of VPC IP address range
- **Route Tables**: Define routing rules
- **Internet Gateway**: Internet access for public subnets
- **NAT Gateway**: Outbound internet for private subnets
- **Security Groups**: Virtual firewall
- **Network ACLs**: Subnet-level firewall

### 7.2 Subnets

**Public Subnet**:
- Has route to Internet Gateway
- Instances can have public IPs
- Direct internet access

**Private Subnet**:
- No direct internet access
- Uses NAT Gateway for outbound
- More secure

**Example**:
```
VPC: 10.0.0.0/16
├── Public Subnet: 10.0.1.0/24 (AZ-1)
│   └── EC2 Instance (Public IP)
│
├── Private Subnet: 10.0.2.0/24 (AZ-1)
│   └── EC2 Instance (Private IP only)
│
├── Public Subnet: 10.0.3.0/24 (AZ-2)
│   └── EC2 Instance (Public IP)
│
└── Private Subnet: 10.0.4.0/24 (AZ-2)
    └── EC2 Instance (Private IP only)
```

### 7.3 IP Addressing

**Private IP Address**:
- Assigned from subnet CIDR
- Used for internal communication
- Cannot be accessed from internet
- Example: 10.0.1.50

**Public IP Address**:
- Routable on internet
- Assigned automatically or manually
- Changes on stop/start (unless Elastic IP)
- Example: 54.123.45.67

**Elastic IP Address**:
- Static public IP address
- Associated with your account
- Can be attached/detached from instances
- Retained when instance stops
- Small charge if not attached

### 7.4 Enhanced Networking

**Benefits**:
- Higher bandwidth (up to 100 Gbps)
- Lower latency
- Lower jitter
- Better packet per second (PPS) performance

**Types**:
- **SR-IOV**: Single Root I/O Virtualization
- **ENA**: Elastic Network Adapter

**Requirements**:
- Supported instance types
- HVM AMI
- Appropriate drivers

---

## 8. Security Groups

### 8.1 What are Security Groups?

**Security Groups** are virtual firewalls that control inbound and outbound traffic for EC2 instances.

### 8.2 Key Characteristics

- **Stateful**: Return traffic is automatically allowed
- **Default Deny**: All inbound traffic denied by default
- **Default Allow**: All outbound traffic allowed by default
- **Instance-Level**: Applied to instances, not subnets
- **Multiple Groups**: Instance can have multiple security groups

### 8.3 Security Group Rules

**Rule Components**:
- **Type**: Protocol (TCP, UDP, ICMP, etc.)
- **Protocol**: Port number or range
- **Source/Destination**: IP address, CIDR block, or another security group
- **Description**: Optional description

**Example Rules**:
```
Inbound Rules:
┌──────────┬──────────┬──────────────┬─────────────┐
│ Type     │ Protocol │ Source       │ Description │
├──────────┼──────────┼──────────────┼─────────────┤
│ SSH      │ TCP 22   │ 0.0.0.0/0   │ SSH access  │
│ HTTP     │ TCP 80    │ 0.0.0.0/0   │ Web traffic │
│ HTTPS    │ TCP 443   │ 0.0.0.0/0   │ Secure web  │
│ Custom   │ TCP 8080  │ sg-12345678 │ App server  │
└──────────┴──────────┴──────────────┴─────────────┘

Outbound Rules:
┌──────────┬──────────┬──────────────┬─────────────┐
│ Type     │ Protocol │ Destination  │ Description │
├──────────┼──────────┼──────────────┼─────────────┤
│ All      │ All      │ 0.0.0.0/0    │ Default     │
└──────────┴──────────┴──────────────┴─────────────┘
```

### 8.4 Security Group Best Practices

**Principle of Least Privilege**:
- Only allow necessary ports
- Restrict source IPs when possible
- Use specific security groups for different roles

**Examples**:
```bash
# Web Server Security Group
- Allow HTTP (80) from ALB security group
- Allow HTTPS (443) from ALB security group
- Allow SSH (22) from bastion security group only

# Database Security Group
- Allow MySQL (3306) from application security group only
- No internet access

# Bastion Host Security Group
- Allow SSH (22) from your office IP only
```

### 8.5 Security Groups vs Network ACLs

| Feature | Security Groups | Network ACLs |
|---------|----------------|--------------|
| **Level** | Instance level | Subnet level |
| **Stateful** | Yes | No |
| **Rules** | Allow only | Allow/Deny |
| **Evaluation** | All rules evaluated | Rules evaluated in order |
| **Default** | Deny inbound, Allow outbound | Deny all |

---

## 9. Key Pairs

### 9.1 What are Key Pairs?

**Key Pairs** are used to securely connect to EC2 instances:
- **Public Key**: Stored on EC2 instance
- **Private Key**: You keep securely (never share)

### 9.2 Key Pair Types

**RSA**:
- Traditional key type
- Supported by all AMIs
- 1024, 2048, or 4096 bits

**ED25519**:
- Modern, more secure
- Smaller key size
- Better performance
- Supported by newer AMIs

### 9.3 Using Key Pairs

**Linux/Unix**:
```bash
# Generate key pair (if creating locally)
ssh-keygen -t rsa -b 4096 -f my-key

# Connect to instance
ssh -i my-key.pem ec2-user@54.123.45.67

# For Amazon Linux 2
ssh -i my-key.pem ec2-user@54.123.45.67

# For Ubuntu
ssh -i my-key.pem ubuntu@54.123.45.67
```

**Windows (RDP)**:
- Use password (set via Systems Manager or user data)
- Or use AWS Systems Manager Session Manager (no key needed)

### 9.4 Key Pair Best Practices

**Security**:
- Never share private key
- Use different keys for different environments
- Rotate keys regularly
- Store private keys securely (encrypted)

**Management**:
- Use AWS Systems Manager for keyless access
- Consider using AWS Session Manager
- Use IAM roles instead of keys when possible

---

## Summary of Part 1

**Key Takeaways**:
1. **EC2** provides resizable compute capacity in the cloud
2. **Instance Types** are optimized for different workloads (compute, memory, storage, GPU)
3. **AMIs** are templates containing OS and software configuration
4. **Instance Lifecycle** includes states: pending, running, stopping, stopped, terminating, terminated
5. **Storage Options** include EBS (persistent), Instance Store (ephemeral), and EFS (shared)
6. **Networking** is managed through VPCs, subnets, and security groups
7. **Security Groups** act as virtual firewalls at the instance level
8. **Key Pairs** enable secure SSH/RDP access to instances

**Next**: Part 2 will cover advanced topics including Auto Scaling, Load Balancing, Monitoring, Cost Optimization, and Best Practices.

