# AWS EC2 In-Depth Interview Guide - Part 2: Storage, Networking & Security

## Table of Contents
1. [Elastic Block Store (EBS)](#elastic-block-store-ebs)
2. [Instance Store](#instance-store)
3. [Networking Fundamentals](#networking-fundamentals)
4. [Security Groups](#security-groups)
5. [Network ACLs](#network-acls)
6. [Key Pairs](#key-pairs)
7. [Elastic IP Addresses](#elastic-ip-addresses)
8. [Placement Groups](#placement-groups)

---

## Elastic Block Store (EBS)

### What is EBS?

**Amazon Elastic Block Store (EBS)** provides persistent block storage volumes for EC2 instances. EBS volumes are:
- **Persistent**: Data persists independently of instance lifecycle
- **Network-attached**: Not physically attached to instance hardware
- **Replicated**: Automatically replicated within Availability Zone
- **Backed up**: Can create snapshots for backup/restore

### EBS Volume Types

#### 1. **General Purpose SSD (gp3)**

**Characteristics:**
- Latest generation general-purpose SSD
- Baseline: 3,000 IOPS, 125 MB/s throughput
- Can provision up to 16,000 IOPS and 1,000 MB/s
- **Cost**: ~$0.08/GB-month
- **Use Cases**: Boot volumes, small to medium databases, dev/test environments

**Hands-On Example:**

```bash
# Create gp3 volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type gp3 \
  --iops 3000 \
  --throughput 125 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=MyGP3Volume}]'

# Attach volume to instance
aws ec2 attach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/xvdf
```

#### 2. **General Purpose SSD (gp2)**

**Characteristics:**
- Previous generation (legacy)
- IOPS scales with volume size (3 IOPS/GB, up to 16,000 IOPS)
- Burst capability for small volumes
- **Cost**: ~$0.10/GB-month
- **Use Cases**: Boot volumes, small databases

**IOPS Calculation:**
- Minimum: 100 IOPS (for volumes < 33.33 GB)
- Baseline: 3 IOPS per GB
- Maximum: 16,000 IOPS (for volumes ≥ 5,334 GB)
- Burst: Up to 3,000 IOPS for volumes < 1,000 GB

```bash
# Create gp2 volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type gp2 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=MyGP2Volume}]'
```

#### 3. **Provisioned IOPS SSD (io1/io2)**

**Characteristics:**
- **io1**: Previous generation
- **io2**: Latest generation (99.999% durability)
- Provision IOPS independently of volume size
- **io1**: Up to 64,000 IOPS, 1,000 MB/s
- **io2**: Up to 256,000 IOPS, 4,000 MB/s
- **Cost**: ~$0.125/GB-month + $0.065/provisioned IOPS-month
- **Use Cases**: Critical databases, I/O-intensive applications

**IOPS to Volume Size Ratio:**
- **io1**: Up to 50:1 (50 IOPS per GB)
- **io2**: Up to 500:1 (500 IOPS per GB)

```bash
# Create io2 volume with high IOPS
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type io2 \
  --iops 10000 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=MyIO2Volume}]'
```

#### 4. **Throughput Optimized HDD (st1)**

**Characteristics:**
- Low-cost HDD for frequently accessed, throughput-intensive workloads
- Baseline: 40 MB/s per TB, up to 500 MB/s
- **Cost**: ~$0.045/GB-month
- **Use Cases**: Big data, data warehouses, log processing
- **Cannot be boot volume**

```bash
# Create st1 volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 500 \
  --volume-type st1 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=MyST1Volume}]'
```

#### 5. **Cold HDD (sc1)**

**Characteristics:**
- Lowest-cost HDD for less frequently accessed workloads
- Baseline: 12 MB/s per TB, up to 250 MB/s
- **Cost**: ~$0.015/GB-month
- **Use Cases**: Throughput-oriented storage for large volumes of infrequently accessed data
- **Cannot be boot volume**

```bash
# Create sc1 volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 1000 \
  --volume-type sc1 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=MySC1Volume}]'
```

### EBS Volume Comparison

| Volume Type | IOPS | Throughput | Use Case | Cost/GB |
|------------|------|------------|----------|---------|
| gp3 | 3,000-16,000 | 125-1,000 MB/s | General purpose | $0.08 |
| gp2 | 100-16,000 | 125-250 MB/s | General purpose | $0.10 |
| io1 | Up to 64,000 | Up to 1,000 MB/s | Critical DBs | $0.125 + IOPS |
| io2 | Up to 256,000 | Up to 4,000 MB/s | Critical DBs | $0.125 + IOPS |
| st1 | 500 MB/s | Throughput-intensive | Big data | $0.045 |
| sc1 | 250 MB/s | Infrequent access | Archive | $0.015 |

### EBS Snapshots

**What are Snapshots?**
- Point-in-time backups of EBS volumes
- Stored in S3 (but not visible in S3 console)
- Incremental: Only changed blocks are saved
- Can create AMI from snapshot
- Can copy snapshots across regions

**Hands-On: Snapshot Management**

```bash
# Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-0123456789abcdef0 \
  --description "Backup before upgrade" \
  --tag-specifications 'ResourceType=snapshot,Tags=[{Key=Name,Value=MySnapshot}]'

# List snapshots
aws ec2 describe-snapshots \
  --owner-ids self \
  --query 'Snapshots[*].[SnapshotId,VolumeId,StartTime,State]' \
  --output table

# Create volume from snapshot
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --snapshot-id snap-0123456789abcdef0 \
  --volume-type gp3 \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=RestoredVolume}]'

# Copy snapshot to another region
aws ec2 copy-snapshot \
  --source-region us-east-1 \
  --source-snapshot-id snap-0123456789abcdef0 \
  --description "Copied snapshot" \
  --region us-west-2

# Delete snapshot
aws ec2 delete-snapshot --snapshot-id snap-0123456789abcdef0
```

### EBS Volume Operations

#### Attach/Detach Volumes

```bash
# Attach volume
aws ec2 attach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device /dev/xvdf

# Detach volume (must unmount in OS first)
aws ec2 detach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-1234567890abcdef0

# Force detach (if instance is stopped/terminated)
aws ec2 detach-volume \
  --volume-id vol-0123456789abcdef0 \
  --force
```

#### Modify Volume Attributes

```bash
# Increase volume size
aws ec2 modify-volume \
  --volume-id vol-0123456789abcdef0 \
  --size 200

# Modify volume type (requires downtime)
# Step 1: Stop instance
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
aws ec2 wait instance-stopped --instance-ids i-1234567890abcdef0

# Step 2: Modify volume type
aws ec2 modify-volume \
  --volume-id vol-0123456789abcdef0 \
  --volume-type io2 \
  --iops 5000

# Step 3: Start instance
aws ec2 start-instances --instance-ids i-1234567890abcdef0

# Modify IOPS (gp3 only)
aws ec2 modify-volume \
  --volume-id vol-0123456789abcdef0 \
  --iops 5000 \
  --throughput 250
```

#### Encrypt EBS Volumes

```bash
# Create encrypted volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type gp3 \
  --encrypted \
  --kms-key-id alias/aws/ebs \
  --tag-specifications 'ResourceType=volume,Tags=[{Key=Name,Value=EncryptedVolume}]'

# Encrypt existing volume (requires snapshot)
# Step 1: Create snapshot
aws ec2 create-snapshot \
  --volume-id vol-0123456789abcdef0 \
  --description "Snapshot for encryption"

# Step 2: Copy snapshot with encryption
aws ec2 copy-snapshot \
  --source-region us-east-1 \
  --source-snapshot-id snap-0123456789abcdef0 \
  --encrypted \
  --kms-key-id alias/aws/ebs

# Step 3: Create encrypted volume from snapshot
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --snapshot-id snap-0987654321fedcba0 \
  --encrypted
```

### EBS Multi-Attach (io1/io2 only)

**Use Case**: Attach same volume to multiple instances in same AZ

```bash
# Enable multi-attach when creating volume
aws ec2 create-volume \
  --availability-zone us-east-1a \
  --size 100 \
  --volume-type io1 \
  --iops 3000 \
  --multi-attach-enabled

# Attach to multiple instances
aws ec2 attach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-instance1 \
  --device /dev/xvdf

aws ec2 attach-volume \
  --volume-id vol-0123456789abcdef0 \
  --instance-id i-instance2 \
  --device /dev/xvdf
```

---

## Instance Store

### What is Instance Store?

**Instance Store** (also called ephemeral storage) provides:
- **Temporary block-level storage**
- **Physically attached** to the host computer
- **Data is lost** when instance stops/terminates
- **Higher performance** than EBS for some workloads
- **No additional cost** (included with instance)

### Instance Store vs EBS

| Feature | Instance Store | EBS |
|--------|----------------|-----|
| Persistence | Ephemeral (lost on stop/terminate) | Persistent |
| Performance | Very high (local SSD) | Network-attached |
| Cost | Included with instance | Additional cost |
| Size | Fixed per instance type | Configurable |
| Backup | Manual (copy to EBS/S3) | Snapshots |
| Use Case | Cache, temp data, swap | Boot volumes, databases |

### Instance Store Instance Types

**Examples:**
- `i3.large`: 1x 475 GB NVMe SSD
- `i3.xlarge`: 1x 950 GB NVMe SSD
- `d2.xlarge`: 3x 2,000 GB HDD
- `m5d.large`: 1x 75 GB NVMe SSD

### Hands-On: Using Instance Store

```bash
# Launch instance with instance store
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type i3.large \
  --key-name my-key-pair \
  --security-group-ids sg-0123456789abcdef0 \
  --subnet-id subnet-0123456789abcdef0

# After SSH into instance, check instance store
# For i3.large, instance store is typically /dev/nvme0n1
lsblk
sudo file -s /dev/nvme0n1
sudo mkfs -t xfs /dev/nvme0n1
sudo mkdir /mnt/instance-store
sudo mount /dev/nvme0n1 /mnt/instance-store
```

---

## Networking Fundamentals

### VPC (Virtual Private Cloud)

**What is VPC?**
- Isolated virtual network in AWS
- Complete control over networking
- Custom IP address ranges
- Subnets, route tables, internet gateways
- Security groups and network ACLs

### Key Networking Components

#### 1. **Subnets**

**Public Subnet:**
- Has route to Internet Gateway
- Instances can have public IPs
- Use for: Web servers, load balancers

**Private Subnet:**
- No direct route to Internet Gateway
- Instances cannot have public IPs
- Use for: Databases, application servers

```bash
# Create subnet
aws ec2 create-subnet \
  --vpc-id vpc-0123456789abcdef0 \
  --cidr-block 10.0.1.0/24 \
  --availability-zone us-east-1a \
  --tag-specifications 'ResourceType=subnet,Tags=[{Key=Name,Value=PublicSubnet}]'
```

#### 2. **Internet Gateway (IGW)**

**Purpose**: Allows communication between VPC and internet

```bash
# Create and attach Internet Gateway
aws ec2 create-internet-gateway \
  --tag-specifications 'ResourceType=internet-gateway,Tags=[{Key=Name,Value=MyIGW}]'

aws ec2 attach-internet-gateway \
  --internet-gateway-id igw-0123456789abcdef0 \
  --vpc-id vpc-0123456789abcdef0
```

#### 3. **NAT Gateway**

**Purpose**: Allows private subnet instances to access internet (outbound only)

```bash
# Create NAT Gateway (requires Elastic IP)
aws ec2 allocate-address --domain vpc

aws ec2 create-nat-gateway \
  --subnet-id subnet-0123456789abcdef0 \
  --allocation-id eipalloc-0123456789abcdef0 \
  --tag-specifications 'ResourceType=natgateway,Tags=[{Key=Name,Value=MyNATGateway}]'
```

#### 4. **Route Tables**

```bash
# Create route table
aws ec2 create-route-table \
  --vpc-id vpc-0123456789abcdef0 \
  --tag-specifications 'ResourceType=route-table,Tags=[{Key=Name,Value=PublicRouteTable}]'

# Add route to Internet Gateway
aws ec2 create-route \
  --route-table-id rtb-0123456789abcdef0 \
  --destination-cidr-block 0.0.0.0/0 \
  --gateway-id igw-0123456789abcdef0

# Associate route table with subnet
aws ec2 associate-route-table \
  --subnet-id subnet-0123456789abcdef0 \
  --route-table-id rtb-0123456789abcdef0
```

### Network Interfaces

**Elastic Network Interface (ENI):**
- Virtual network interface
- Can attach/detach from instances
- Can have multiple IP addresses
- Can move between instances

```bash
# Create network interface
aws ec2 create-network-interface \
  --subnet-id subnet-0123456789abcdef0 \
  --description "Secondary ENI" \
  --groups sg-0123456789abcdef0 \
  --private-ip-address 10.0.1.50

# Attach ENI to instance
aws ec2 attach-network-interface \
  --network-interface-id eni-0123456789abcdef0 \
  --instance-id i-1234567890abcdef0 \
  --device-index 1
```

---

## Security Groups

### What are Security Groups?

**Security Groups** are virtual firewalls that control:
- **Inbound traffic** (ingress)
- **Outbound traffic** (egress)
- **Stateful**: Return traffic is automatically allowed
- **Default deny**: All traffic denied by default
- **Instance-level**: Applied to instances

### Security Group Rules

**Rule Components:**
- **Type**: Protocol (TCP, UDP, ICMP, etc.)
- **Port Range**: Port or port range
- **Source/Destination**: CIDR block, security group, or prefix list
- **Description**: Optional description

### Default Security Group

**Default Rules:**
- **Inbound**: None (all denied)
- **Outbound**: All traffic allowed (0.0.0.0/0)

### Hands-On: Security Group Management

#### Create Security Group

```bash
# Create security group
aws ec2 create-security-group \
  --group-name web-server-sg \
  --description "Security group for web servers" \
  --vpc-id vpc-0123456789abcdef0 \
  --tag-specifications 'ResourceType=security-group,Tags=[{Key=Name,Value=WebServerSG}]'
```

#### Add Inbound Rules

```bash
# Allow HTTP from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0 \
  --group-name web-server-sg

# Allow HTTPS from anywhere
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0

# Allow SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 22 \
  --cidr 203.0.113.0/24

# Allow traffic from another security group
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 3306 \
  --source-group sg-0987654321fedcba0 \
  --group-owner-id 123456789012

# Allow ICMP (ping)
aws ec2 authorize-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol icmp \
  --port -1 \
  --cidr 10.0.0.0/16
```

#### Add Outbound Rules

```bash
# Allow all outbound traffic (default, but can restrict)
aws ec2 authorize-security-group-egress \
  --group-id sg-0123456789abcdef0 \
  --protocol -1 \
  --cidr 0.0.0.0/0

# Restrict outbound to specific port
aws ec2 authorize-security-group-egress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

#### Remove Rules

```bash
# Remove inbound rule
aws ec2 revoke-security-group-ingress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0

# Remove outbound rule
aws ec2 revoke-security-group-egress \
  --group-id sg-0123456789abcdef0 \
  --protocol tcp \
  --port 443 \
  --cidr 0.0.0.0/0
```

#### Describe Security Groups

```bash
# List all security groups
aws ec2 describe-security-groups \
  --query 'SecurityGroups[*].[GroupId,GroupName,Description]' \
  --output table

# Get specific security group details
aws ec2 describe-security-groups \
  --group-ids sg-0123456789abcdef0

# Get security groups for instance
aws ec2 describe-instances \
  --instance-ids i-1234567890abcdef0 \
  --query 'Reservations[0].Instances[0].SecurityGroups' \
  --output table
```

### Security Group Best Practices

1. **Principle of Least Privilege**: Only allow necessary ports
2. **Use Specific IPs**: Avoid 0.0.0.0/0 when possible
3. **Reference Security Groups**: Use security group references for internal traffic
4. **Separate by Tier**: Different security groups for web, app, database tiers
5. **Regular Audits**: Review and remove unused rules
6. **Document Rules**: Use descriptions for rule purpose

### Common Security Group Configurations

#### Web Server Security Group

```bash
# HTTP (80) - Public
# HTTPS (443) - Public
# SSH (22) - Specific IPs only
# No database ports exposed
```

#### Database Security Group

```bash
# MySQL (3306) - Only from app servers
# PostgreSQL (5432) - Only from app servers
# No public access
# No SSH
```

#### Application Server Security Group

```bash
# Custom app port (8080) - Only from load balancer
# SSH (22) - Specific IPs only
# Outbound to database security group
```

---

## Network ACLs

### What are Network ACLs?

**Network ACLs (NACLs)** are:
- **Stateless**: Must allow both inbound and outbound
- **Subnet-level**: Applied to entire subnet
- **Rule numbers**: Processed in order (lower numbers first)
- **Default allow**: Default NACL allows all traffic

### NACL vs Security Group

| Feature | NACL | Security Group |
|---------|------|----------------|
| Level | Subnet | Instance |
| Stateful | No (stateless) | Yes (stateful) |
| Default | Allow all | Deny all |
| Rules | Explicit allow/deny | Allow only |
| Evaluation | Rule number order | All rules evaluated |

### Hands-On: Network ACL Management

```bash
# Create custom network ACL
aws ec2 create-network-acl \
  --vpc-id vpc-0123456789abcdef0 \
  --tag-specifications 'ResourceType=network-acl,Tags=[{Key=Name,Value=MyNACL}]'

# Create inbound rule (allow HTTP)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0123456789abcdef0 \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=80,To=80 \
  --cidr-block 0.0.0.0/0 \
  --egress false \
  --rule-action allow

# Create outbound rule (allow response)
aws ec2 create-network-acl-entry \
  --network-acl-id acl-0123456789abcdef0 \
  --rule-number 100 \
  --protocol tcp \
  --port-range From=1024,To=65535 \
  --cidr-block 0.0.0.0/0 \
  --egress true \
  --rule-action allow

# Associate NACL with subnet
aws ec2 replace-network-acl-association \
  --association-id aclassoc-0123456789abcdef0 \
  --network-acl-id acl-0123456789abcdef0
```

---

## Key Pairs

### What are Key Pairs?

**Key Pairs** are used for:
- **SSH access** to Linux instances
- **RDP access** to Windows instances (optional)
- **Public key** stored on instance
- **Private key** downloaded and stored securely

### Key Pair Types

1. **RSA**: Traditional, widely supported
2. **ED25519**: Modern, more secure, faster

### Hands-On: Key Pair Management

#### Create Key Pair

```bash
# Create key pair (AWS generates and returns private key)
aws ec2 create-key-pair \
  --key-name my-key-pair \
  --key-type rsa \
  --key-format pem \
  --query 'KeyMaterial' \
  --output text > my-key-pair.pem

# Set permissions (Linux/Mac)
chmod 400 my-key-pair.pem

# Create ED25519 key pair
aws ec2 create-key-pair \
  --key-name my-ed25519-key \
  --key-type ed25519 \
  --query 'KeyMaterial' \
  --output text > my-ed25519-key.pem
chmod 400 my-ed25519-key.pem
```

#### Import Existing Key Pair

```bash
# If you have existing public key
aws ec2 import-key-pair \
  --key-name my-imported-key \
  --public-key-material fileb://~/.ssh/id_rsa.pub
```

#### List Key Pairs

```bash
# List all key pairs
aws ec2 describe-key-pairs \
  --query 'KeyPairs[*].[KeyName,KeyType,KeyFingerprint]' \
  --output table
```

#### Delete Key Pair

```bash
# Delete key pair (doesn't affect running instances)
aws ec2 delete-key-pair --key-name my-key-pair
```

#### SSH to Instance

```bash
# SSH using key pair
ssh -i my-key-pair.pem ec2-user@54.123.45.67

# For Amazon Linux 2
ssh -i my-key-pair.pem ec2-user@<public-ip>

# For Ubuntu
ssh -i my-key-pair.pem ubuntu@<public-ip>

# For RHEL/CentOS
ssh -i my-key-pair.pem ec2-user@<public-ip>
```

---

## Elastic IP Addresses

### What are Elastic IPs?

**Elastic IP (EIP)** addresses are:
- **Static public IP** addresses
- **Persistent**: Don't change when instance stops/starts
- **Region-specific**: Allocated per region
- **Free when attached**: No charge if attached to running instance
- **Charged when idle**: $0.005/hour if not attached

### Hands-On: Elastic IP Management

#### Allocate Elastic IP

```bash
# Allocate Elastic IP
aws ec2 allocate-address \
  --domain vpc \
  --tag-specifications 'ResourceType=elastic-ip,Tags=[{Key=Name,Value=MyEIP}]'

# Allocate for EC2-Classic (legacy)
aws ec2 allocate-address --domain standard
```

#### Associate Elastic IP with Instance

```bash
# Associate Elastic IP
aws ec2 associate-address \
  --instance-id i-1234567890abcdef0 \
  --allocation-id eipalloc-0123456789abcdef0

# Associate with network interface
aws ec2 associate-address \
  --network-interface-id eni-0123456789abcdef0 \
  --allocation-id eipalloc-0123456789abcdef0 \
  --private-ip-address 10.0.1.50
```

#### Disassociate Elastic IP

```bash
# Disassociate Elastic IP
aws ec2 disassociate-address \
  --association-id eipassoc-0123456789abcdef0
```

#### Release Elastic IP

```bash
# Release Elastic IP (charges apply if not attached)
aws ec2 release-address \
  --allocation-id eipalloc-0123456789abcdef0
```

#### List Elastic IPs

```bash
# List all Elastic IPs
aws ec2 describe-addresses \
  --query 'Addresses[*].[PublicIp,AllocationId,InstanceId,AssociationId]' \
  --output table
```

### Elastic IP Best Practices

1. **Attach to Running Instances**: Avoid charges for idle EIPs
2. **Use for Critical Services**: Web servers, databases that need static IPs
3. **Release Unused EIPs**: Don't keep idle EIPs
4. **Use with Load Balancers**: Consider using ALB/NLB instead for high availability

---

## Placement Groups

### What are Placement Groups?

**Placement Groups** control instance placement for:
- **Low latency**: Cluster placement
- **High availability**: Spread placement
- **Partition placement**: For distributed systems

### Placement Group Types

#### 1. **Cluster Placement Group**

**Characteristics:**
- Instances in same AZ
- Low latency, high throughput
- Single point of failure (same rack)
- **Use Cases**: HPC, tightly coupled workloads

```bash
# Create cluster placement group
aws ec2 create-placement-group \
  --group-name my-cluster-pg \
  --strategy cluster

# Launch instances in placement group
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type c5.xlarge \
  --placement GroupName=my-cluster-pg \
  --count 4
```

#### 2. **Spread Placement Group**

**Characteristics:**
- Instances across distinct hardware
- Maximum availability
- Limited to 7 instances per AZ
- **Use Cases**: Critical applications, small clusters

```bash
# Create spread placement group
aws ec2 create-placement-group \
  --group-name my-spread-pg \
  --strategy spread

# Launch instances in spread placement group
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type t3.micro \
  --placement GroupName=my-spread-pg \
  --count 3
```

#### 3. **Partition Placement Group**

**Characteristics:**
- Instances in logical partitions
- Up to 7 partitions per AZ
- Multiple instances per partition
- **Use Cases**: Distributed systems (HDFS, Cassandra, Kafka)

```bash
# Create partition placement group
aws ec2 create-placement-group \
  --group-name my-partition-pg \
  --strategy partition

# Launch instances in partition placement group
aws ec2 run-instances \
  --image-id ami-0c55b159cbfafe1f0 \
  --instance-type m5.large \
  --placement GroupName=my-partition-pg,PartitionNumber=1 \
  --count 2
```

### Placement Group Rules

1. **Cannot merge**: Can't move instances between groups
2. **Same instance type**: Cluster groups require same instance type
3. **AZ restrictions**: Cluster groups must be in same AZ
4. **Reserved Instances**: Can't specify placement group for RIs

---

## Interview Questions & Answers

### Q1: What's the difference between EBS and Instance Store?

**Answer:**
- **EBS**: Persistent, network-attached, survives instance termination, additional cost, can detach/attach
- **Instance Store**: Ephemeral, physically attached, lost on termination, included in cost, cannot detach

### Q2: When would you use gp3 vs io2?

**Answer:**
- **gp3**: General purpose workloads, cost-effective, up to 16,000 IOPS
- **io2**: Critical databases, need >16,000 IOPS, need 99.999% durability, up to 256,000 IOPS

### Q3: What's the difference between Security Groups and NACLs?

**Answer:**
- **Security Groups**: Instance-level, stateful, default deny, allow rules only
- **NACLs**: Subnet-level, stateless, default allow, explicit allow/deny rules

### Q4: What happens to an Elastic IP when an instance is terminated?

**Answer:**
- If Elastic IP is associated with instance, it's disassociated but **not released**
- You're charged for the Elastic IP if it's not associated with a running instance
- Best practice: Release unused Elastic IPs or associate with new instance

### Q5: Can you attach an EBS volume to multiple instances?

**Answer:**
- **Yes, but only for io1/io2 volumes** with Multi-Attach enabled
- All instances must be in the same AZ
- Use cases: Clustered applications, high availability databases
- Requires file system that supports concurrent writes (e.g., cluster file systems)

### Q6: What's the difference between public and private subnets?

**Answer:**
- **Public Subnet**: Has route to Internet Gateway, instances can have public IPs, used for web servers
- **Private Subnet**: No direct internet access, instances use NAT Gateway for outbound, used for databases

---

## Summary

**Key Takeaways:**
1. **EBS**: Persistent block storage with multiple volume types (gp3, io2, st1, sc1)
2. **Instance Store**: Ephemeral storage with high performance, included with instance
3. **Networking**: VPC, subnets, IGW, NAT Gateway, route tables
4. **Security Groups**: Stateful, instance-level firewall
5. **NACLs**: Stateless, subnet-level firewall
6. **Key Pairs**: SSH access to Linux instances
7. **Elastic IPs**: Static public IP addresses
8. **Placement Groups**: Control instance placement (cluster, spread, partition)

**Next Part**: Part 3 will cover Auto Scaling, Load Balancing, Monitoring, Best Practices, and Advanced Topics.

---

**Part 2 Complete** - Ready for Part 3 on prompt

