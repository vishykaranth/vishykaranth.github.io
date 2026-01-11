# AWS Topics - Quick Reference (One-Liner Each)

## Compute Services

- **EC2 (Elastic Compute Cloud)**: Virtual servers in the cloud with scalable compute capacity.
- **Lambda**: Serverless compute service that runs code in response to events without managing servers.
- **ECS (Elastic Container Service)**: Fully managed container orchestration service for Docker containers.
- **EKS (Elastic Kubernetes Service)**: Managed Kubernetes service for running containerized applications.
- **Fargate**: Serverless compute engine for containers that eliminates server management.
- **Elastic Beanstalk**: Platform-as-a-Service (PaaS) for deploying and scaling web applications.
- **Batch**: Fully managed service for running batch computing workloads at any scale.
- **Lightsail**: Simplified virtual private servers with pre-configured application stacks.

## Storage Services

- **S3 (Simple Storage Service)**: Object storage service with unlimited scalability and 99.999999999% durability.
- **EBS (Elastic Block Store)**: Persistent block storage volumes for EC2 instances.
- **EFS (Elastic File System)**: Fully managed, scalable file storage for EC2 instances.
- **FSx**: Fully managed file systems for Windows and Lustre workloads.
- **Storage Gateway**: Hybrid cloud storage service connecting on-premises environments to AWS cloud storage.
- **Glacier**: Low-cost archival storage service for long-term data retention.
- **S3 Intelligent-Tiering**: Automatic cost optimization by moving data between access tiers.

## Database Services

- **RDS (Relational Database Service)**: Managed relational database service supporting MySQL, PostgreSQL, Oracle, SQL Server, MariaDB.
- **DynamoDB**: Fully managed NoSQL database with single-digit millisecond performance at any scale.
- **Aurora**: High-performance managed relational database compatible with MySQL and PostgreSQL.
- **Redshift**: Fully managed data warehouse service for analytics and business intelligence.
- **ElastiCache**: In-memory caching service supporting Redis and Memcached.
- **DocumentDB**: Fully managed MongoDB-compatible document database service.
- **Neptune**: Fully managed graph database service for applications with highly connected data.
- **Timestream**: Fully managed time-series database for IoT and operational applications.
- **QLDB (Quantum Ledger Database)**: Fully managed ledger database with transparent, immutable, and cryptographically verifiable transaction log.

## Networking & Content Delivery

- **VPC (Virtual Private Cloud)**: Isolated virtual network in AWS cloud with complete control over network configuration.
- **CloudFront**: Global content delivery network (CDN) for fast content delivery with low latency.
- **Route 53**: Scalable DNS and domain name registration service with health checking and routing.
- **API Gateway**: Fully managed service for creating, publishing, and managing REST and WebSocket APIs.
- **Direct Connect**: Dedicated network connection from on-premises to AWS for consistent network performance.
- **VPN**: Secure site-to-site or remote access VPN connections to AWS VPC.
- **Transit Gateway**: Network transit hub for connecting VPCs and on-premises networks.
- **PrivateLink**: Private connectivity between VPCs and AWS services without internet exposure.
- **Global Accelerator**: Network service that improves availability and performance of applications for global users.

## Security & Identity

- **IAM (Identity and Access Management)**: Service for securely controlling access to AWS resources and services.
- **Cognito**: User identity and data synchronization service for web and mobile applications.
- **Secrets Manager**: Service for storing and rotating secrets, API keys, and database credentials.
- **KMS (Key Management Service)**: Managed service for creating and controlling encryption keys.
- **CloudHSM**: Dedicated hardware security module (HSM) for managing encryption keys.
- **WAF (Web Application Firewall)**: Firewall service protecting web applications from common exploits.
- **Shield**: Managed DDoS protection service for applications running on AWS.
- **GuardDuty**: Threat detection service that continuously monitors for malicious activity.
- **Security Hub**: Centralized security management service aggregating security findings from multiple AWS services.
- **Macie**: Security service that uses machine learning to discover and protect sensitive data in S3.
- **Inspector**: Automated security assessment service for applications deployed on EC2.
- **Artifact**: On-demand access to AWS compliance reports and certifications.

## Monitoring & Management

- **CloudWatch**: Monitoring and observability service for AWS resources and applications.
- **CloudTrail**: Service that logs API calls and user activity for governance, compliance, and auditing.
- **Config**: Service that records configuration changes and evaluates resource configurations for compliance.
- **Systems Manager**: Operations management service for managing and configuring AWS resources at scale.
- **CloudFormation**: Infrastructure as Code service for modeling and provisioning AWS resources.
- **OpsWorks**: Configuration management service using Chef and Puppet for automated deployments.
- **Trusted Advisor**: Service that analyzes AWS environment and provides recommendations for cost optimization and security.
- **Service Catalog**: Service for creating and managing catalogs of IT services approved for use on AWS.
- **Control Tower**: Service for setting up and governing secure, compliant multi-account AWS environments.
- **Well-Architected Tool**: Tool for reviewing workloads against AWS Well-Architected Framework best practices.

## Application Integration

- **SQS (Simple Queue Service)**: Fully managed message queuing service for decoupling and scaling microservices.
- **SNS (Simple Notification Service)**: Fully managed pub/sub messaging service for distributed systems.
- **EventBridge**: Serverless event bus service for connecting applications using events.
- **Step Functions**: Serverless orchestration service for coordinating multiple AWS services into serverless workflows.
- **AppSync**: Fully managed GraphQL service for building data-driven applications with real-time and offline capabilities.
- **MQ**: Managed message broker service for Apache ActiveMQ and RabbitMQ.

## Analytics

- **Athena**: Serverless interactive query service for analyzing data in S3 using standard SQL.
- **EMR (Elastic MapReduce)**: Big data platform for processing large amounts of data using Hadoop, Spark, and other frameworks.
- **Kinesis**: Platform for streaming data in real-time for analytics, machine learning, and application integration.
- **QuickSight**: Business intelligence service for building visualizations and dashboards.
- **Data Pipeline**: Web service for orchestrating data movement and transformation between AWS services.
- **Glue**: Fully managed ETL service for preparing and transforming data for analytics.
- **Lake Formation**: Service for building, securing, and managing data lakes.

## Machine Learning

- **SageMaker**: Fully managed service for building, training, and deploying machine learning models.
- **Comprehend**: Natural language processing service for extracting insights and relationships from text.
- **Rekognition**: Deep learning-based image and video analysis service for object and scene detection.
- **Polly**: Text-to-speech service that converts text into lifelike speech.
- **Translate**: Neural machine translation service supporting multiple languages.
- **Transcribe**: Automatic speech recognition service for converting speech to text.
- **Lex**: Service for building conversational interfaces using voice and text.
- **Forecast**: Fully managed service for generating accurate forecasts using machine learning.
- **Personalize**: Machine learning service for creating personalized recommendations.
- **Textract**: Service for extracting text and data from scanned documents using machine learning.

## Developer Tools

- **CodeCommit**: Fully managed source control service hosting private Git repositories.
- **CodeBuild**: Fully managed build service for compiling source code and running tests.
- **CodeDeploy**: Automated deployment service for updating applications on EC2, Lambda, and on-premises servers.
- **CodePipeline**: Fully managed continuous delivery service for automating release pipelines.
- **CodeStar**: Cloud-based development service for building, testing, and deploying applications on AWS.
- **Cloud9**: Cloud-based integrated development environment (IDE) for writing, running, and debugging code.
- **X-Ray**: Service for analyzing and debugging distributed applications, including microservices.

## Migration & Transfer

- **Migration Hub**: Central location for tracking migration progress across multiple AWS and partner solutions.
- **Database Migration Service (DMS)**: Service for migrating databases to AWS with minimal downtime.
- **Server Migration Service (SMS)**: Agentless service for migrating on-premises servers to AWS.
- **Application Discovery Service**: Service for automatically discovering on-premises applications and dependencies.
- **DataSync**: Online data transfer service for moving large amounts of data to and from AWS.
- **Snowball**: Petabyte-scale data transport solution using physical storage devices.
- **Snowmobile**: Exabyte-scale data transfer service using a 45-foot shipping container.

## Cost Management

- **Cost Explorer**: Service for visualizing, understanding, and managing AWS costs and usage over time.
- **Budgets**: Service for setting custom cost and usage budgets with automated alerts.
- **Reserved Instances**: Billing discount for committing to use EC2 instances for 1 or 3 years.
- **Savings Plans**: Flexible pricing model providing savings on AWS compute usage.
- **Spot Instances**: Unused EC2 capacity available at up to 90% discount compared to On-Demand prices.
- **Cost and Usage Report**: Detailed billing report containing line items for each AWS service.

## Architecture & Best Practices

- **Well-Architected Framework**: Framework for building secure, high-performing, resilient, and efficient cloud architectures.
- **Multi-Account Strategy**: Best practices for organizing AWS accounts for security, compliance, and billing.
- **Disaster Recovery**: Strategies for RTO/RPO including backup/restore, pilot light, warm standby, and multi-site.
- **High Availability**: Designing systems for 99.99%+ uptime with redundancy and failover mechanisms.
- **Auto Scaling**: Automatically adjusting compute capacity to maintain performance and minimize costs.
- **Load Balancing**: Distributing incoming application traffic across multiple targets for high availability.
- **Infrastructure as Code**: Managing infrastructure through code using CloudFormation, Terraform, or CDK.
- **Serverless Architecture**: Building applications without managing servers using Lambda, API Gateway, and other services.
- **Microservices**: Architectural pattern for building applications as collections of loosely coupled services.
- **Event-Driven Architecture**: Building applications that respond to events in real-time using EventBridge, SQS, SNS.

## Compliance & Governance

- **Compliance Programs**: Understanding SOC, PCI-DSS, HIPAA, GDPR, and other compliance frameworks.
- **Shared Responsibility Model**: Understanding security responsibilities between AWS and customers.
- **Resource Tagging**: Best practices for organizing and managing AWS resources using tags.
- **Organizations**: Service for centrally managing and governing multiple AWS accounts.
- **Service Control Policies**: Policies for managing permissions across multiple AWS accounts.
- **Access Analyzer**: Service for identifying resources shared with external entities.

## Advanced Topics

- **VPC Peering**: Connecting two VPCs for private communication between resources.
- **NAT Gateway**: Managed network address translation service for enabling outbound internet access from private subnets.
- **Elastic IP**: Static IPv4 address designed for dynamic cloud computing.
- **Placement Groups**: Logical grouping of instances for low latency and high network throughput.
- **Dedicated Hosts**: Physical EC2 servers dedicated for your use with visibility into socket and core-level utilization.
- **Elastic Fabric Adapter (EFA)**: Network interface for HPC applications requiring low-latency network performance.
- **Transit VPC**: Architecture pattern for connecting multiple VPCs and on-premises networks.
- **Private Subnets**: Subnets without direct internet access, typically using NAT Gateway for outbound traffic.
- **Public Subnets**: Subnets with direct internet access via Internet Gateway.
- **Security Groups**: Virtual firewall controlling inbound and outbound traffic for EC2 instances.
- **NACLs (Network ACLs)**: Optional layer of security acting as firewall for controlling traffic in and out of subnets.
- **Flow Logs**: Feature enabling capture of information about IP traffic going to and from network interfaces in VPC.

## Serverless & Modern Architecture

- **Serverless Computing**: Running code without provisioning or managing servers, paying only for compute time.
- **Container Services**: ECS, EKS, Fargate for running containerized applications.
- **Event-Driven Computing**: Architecture pattern using events to trigger and communicate between services.
- **API-First Architecture**: Designing applications with APIs as the primary interface.
- **CQRS (Command Query Responsibility Segregation)**: Pattern separating read and write operations for better scalability.
- **Event Sourcing**: Storing all changes as a sequence of events for complete audit trail and time travel.

## Data & Analytics Patterns

- **Data Lake**: Centralized repository for storing structured and unstructured data at any scale.
- **Data Warehouse**: Centralized repository for structured data optimized for analytics and reporting.
- **ETL/ELT**: Extract, Transform, Load patterns for data integration and processing.
- **Stream Processing**: Real-time processing of continuous data streams using Kinesis.
- **Batch Processing**: Processing large volumes of data in scheduled batches using EMR or Batch.
- **Data Pipeline**: Automated workflows for moving and transforming data between systems.

## Disaster Recovery & Backup

- **Backup**: Service for centralizing and automating data protection across AWS services.
- **Snapshot**: Point-in-time copy of EBS volumes or RDS databases for backup and recovery.
- **Cross-Region Replication**: Replicating data across AWS regions for disaster recovery.
- **Multi-AZ Deployment**: Deploying resources across multiple Availability Zones for high availability.
- **Point-in-Time Recovery**: Restoring databases to any point in time within the retention period.

## Performance Optimization

- **Caching Strategies**: Using ElastiCache, CloudFront, and application-level caching for performance.
- **CDN Optimization**: Using CloudFront for global content delivery and reduced latency.
- **Database Optimization**: Tuning RDS, DynamoDB, and other databases for optimal performance.
- **Auto Scaling**: Automatically scaling resources based on demand to maintain performance.
- **Connection Pooling**: Managing database connections efficiently to reduce latency.
- **Read Replicas**: Using read replicas to distribute read traffic and improve performance.

## Cost Optimization

- **Right-Sizing**: Matching instance types and sizes to workload requirements to minimize costs.
- **Reserved Capacity**: Using Reserved Instances and Savings Plans for predictable workloads.
- **Spot Instances**: Using Spot Instances for fault-tolerant and flexible applications.
- **S3 Lifecycle Policies**: Automatically transitioning objects to cheaper storage classes.
- **Idle Resource Detection**: Identifying and terminating unused resources to reduce costs.
- **Cost Allocation Tags**: Using tags to track and optimize costs by department, project, or environment.

## Security Best Practices

- **Least Privilege**: Granting minimum permissions necessary for users and services.
- **Encryption at Rest**: Encrypting data stored in S3, EBS, RDS, and other services.
- **Encryption in Transit**: Using TLS/SSL for encrypting data in transit.
- **Network Segmentation**: Isolating resources using VPCs, subnets, and security groups.
- **Vulnerability Management**: Regularly scanning and patching systems for security vulnerabilities.
- **Incident Response**: Procedures for detecting, responding to, and recovering from security incidents.
- **Audit Logging**: Comprehensive logging of all actions for security and compliance auditing.

## DevOps & CI/CD

- **Continuous Integration**: Automatically building and testing code changes using CodeBuild.
- **Continuous Deployment**: Automatically deploying code changes to production using CodeDeploy.
- **Infrastructure as Code**: Managing infrastructure through version-controlled code.
- **Blue/Green Deployment**: Deployment strategy for zero-downtime updates.
- **Canary Deployment**: Gradually rolling out changes to a subset of users before full deployment.
- **Feature Flags**: Toggling features on/off without code deployment.

## Monitoring & Observability

- **Metrics**: Collecting and analyzing performance metrics using CloudWatch.
- **Logs**: Centralized logging and analysis using CloudWatch Logs.
- **Traces**: Distributed tracing for debugging microservices using X-Ray.
- **Alarms**: Automated notifications for metric thresholds using CloudWatch Alarms.
- **Dashboards**: Visualizing metrics and logs using CloudWatch Dashboards.
- **Anomaly Detection**: Using machine learning to detect unusual patterns in metrics.

## Hybrid Cloud

- **Hybrid Architecture**: Connecting on-premises infrastructure with AWS cloud services.
- **VPN Connections**: Secure connectivity between on-premises and AWS using VPN.
- **Direct Connect**: Dedicated network connection for hybrid cloud scenarios.
- **Storage Gateway**: Bridging on-premises storage with cloud storage.
- **Outposts**: Running AWS infrastructure and services on-premises.

## Edge Computing

- **CloudFront**: Global CDN for edge computing and content delivery.
- **Lambda@Edge**: Running Lambda functions at CloudFront edge locations.
- **Outposts**: Running AWS services at customer premises for low-latency requirements.
- **Wavelength**: Embedding AWS compute and storage at telecommunications carrier edge locations.
- **Local Zones**: Extending AWS infrastructure to metropolitan areas for low-latency applications.

## IoT (Internet of Things)

- **IoT Core**: Managed cloud service for connecting IoT devices to AWS cloud.
- **IoT Device Management**: Onboarding, organizing, monitoring, and remotely managing IoT devices.
- **IoT Analytics**: Service for analyzing IoT device data at scale.
- **IoT Events**: Detecting and responding to events from IoT sensors and applications.
- **Greengrass**: Running local compute, messaging, and data caching for IoT devices.

## Game Development

- **GameLift**: Managed service for deploying, operating, and scaling dedicated game servers.
- **GameSparks**: Backend platform for building, running, and scaling game features.

## Media Services

- **Elemental MediaConvert**: File-based video transcoding service for broadcast and video-on-demand.
- **Elemental MediaLive**: Broadcast-grade live video processing service.
- **Elemental MediaPackage**: Video origination and packaging service for preparing and protecting video for delivery.
- **Elemental MediaStore**: High-performance storage optimized for media.

## Business Applications

- **WorkSpaces**: Managed, secure cloud desktop service for virtual desktops.
- **WorkDocs**: Secure enterprise storage and sharing service with user feedback capabilities.
- **WorkMail**: Secure, managed business email and calendar service.
- **Chime**: Communications service for online meetings, video calls, and business phone systems.
- **Connect**: Cloud-based contact center service for customer engagement.

## Blockchain

- **Managed Blockchain**: Fully managed service for creating and managing blockchain networks.
- **QLDB**: Fully managed ledger database for maintaining complete, verifiable history of data changes.

## Quantum Computing

- **Braket**: Fully managed quantum computing service for exploring and designing quantum algorithms.

## Satellite

- **Ground Station**: Fully managed service for controlling satellite communications, downlinking data, and managing operations.

## RoboMaker

- **RoboMaker**: Cloud service for developing, testing, and deploying intelligent robotics applications.

## AR/VR

- **Sumerian**: Service for creating and running virtual reality, augmented reality, and 3D applications.

## Mobile Services

- **Mobile Hub**: Console for building, testing, and monitoring mobile applications.
- **Device Farm**: App testing service for testing Android, iOS, and web apps on real devices.
- **Pinpoint**: Multichannel marketing communications service for engaging customers.

## End-User Computing

- **AppStream 2.0**: Fully managed application streaming service for delivering desktop applications to users.
- **WorkLink**: Mobile app service for providing secure access to internal websites and web apps.

## Migration & Modernization

- **Application Modernization**: Strategies for modernizing legacy applications using containers, serverless, and microservices.
- **Database Modernization**: Migrating and modernizing databases to cloud-native services.
- **Mainframe Modernization**: Strategies for migrating mainframe workloads to AWS.

## Cost & Billing

- **Pricing Models**: Understanding On-Demand, Reserved Instances, Savings Plans, and Spot pricing.
- **Cost Allocation**: Methods for tracking and allocating costs across departments and projects.
- **Billing Alerts**: Setting up notifications for cost thresholds and budget limits.
- **Cost Optimization Tools**: Using Cost Explorer, Trusted Advisor, and other tools for cost reduction.

## Support & Training

- **Support Plans**: Understanding Basic, Developer, Business, and Enterprise support tiers.
- **Training & Certification**: AWS training programs and certification paths (Solutions Architect, Developer, SysOps, etc.).
- **Documentation**: Comprehensive AWS documentation and best practices guides.
- **Well-Architected Framework**: Five pillars (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization).

---

## Quick Stats

- **Total Services**: 200+ AWS services
- **Global Infrastructure**: 31 regions, 99 availability zones
- **Service Categories**: 20+ major categories
- **Use Cases**: Cloud computing, storage, databases, analytics, machine learning, IoT, and more

---

*This list covers major AWS services and concepts. Each topic can be explored in depth based on your specific needs and use cases.*

