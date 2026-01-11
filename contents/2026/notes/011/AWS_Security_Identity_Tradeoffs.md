# AWS Security & Identity Services - Trade-off Analysis

## Overview

This document provides comprehensive trade-off analysis for AWS Security & Identity services, helping you choose the right service for your specific use case.

---

## 1. IAM (Identity and Access Management)

### What It Is
Service for securely controlling access to AWS resources and services.

### When to Use
- ✅ Managing user access to AWS resources
- ✅ Defining permissions for AWS services
- ✅ Service-to-service authentication
- ✅ Cross-account access
- ✅ Temporary credentials (STS)

### Trade-offs

**Pros**:
- ✅ Centralized access control
- ✅ Fine-grained permissions
- ✅ Integration with all AWS services
- ✅ Free to use
- ✅ Supports MFA
- ✅ Audit trail via CloudTrail

**Cons**:
- ❌ Complex policy language (learning curve)
- ❌ No built-in user directory (use Cognito or external)
- ❌ Policy size limits (6,144 characters)
- ❌ No built-in password management

### Cost
- **Free**: No additional charges
- **Considerations**: Only pay for resources accessed

### Use Cases
- Managing developer access to AWS console
- Granting applications access to S3 buckets
- Cross-account resource sharing
- Service roles for EC2, Lambda, etc.

---

## 2. Cognito

### What It Is
User identity and data synchronization service for web and mobile applications.

### When to Use
- ✅ User authentication for web/mobile apps
- ✅ Social identity providers (Google, Facebook, etc.)
- ✅ User directory management
- ✅ User data synchronization across devices
- ✅ MFA for end users

### Trade-offs

**Pros**:
- ✅ Managed user directory
- ✅ Social identity provider integration
- ✅ Built-in MFA support
- ✅ User data synchronization
- ✅ Scales automatically
- ✅ JWT token generation

**Cons**:
- ❌ Cost per MAU (Monthly Active User)
- ❌ Limited customization compared to self-hosted
- ❌ Vendor lock-in
- ❌ Complex pricing model
- ❌ Limited advanced features vs. Auth0/Okta

### Cost
- **User Pools**: $0.0055 per MAU (first 50K free)
- **Identity Pools**: $0.0055 per MAU
- **Data Sync**: $0.15 per GB transferred

### Use Cases
- Mobile app user authentication
- Web application login
- Social login (Google, Facebook)
- User profile management

### IAM vs Cognito

| Aspect | IAM | Cognito |
|--------|-----|---------|
| **Purpose** | AWS resource access | End-user authentication |
| **Users** | AWS users/roles | Application end users |
| **Scale** | Thousands | Millions |
| **Cost** | Free | Pay per MAU |
| **Use Case** | Internal/admin access | Customer-facing apps |

**Decision**: Use IAM for AWS resource access, Cognito for end-user authentication.

---

## 3. Secrets Manager

### What It Is
Service for storing and rotating secrets, API keys, and database credentials.

### When to Use
- ✅ Database credentials with rotation
- ✅ API keys that need rotation
- ✅ RDS, Redshift, DocumentDB password rotation
- ✅ Secrets accessed by multiple services
- ✅ Audit trail for secret access

### Trade-offs

**Pros**:
- ✅ Automatic secret rotation
- ✅ Built-in rotation for RDS, Redshift
- ✅ Fine-grained access control
- ✅ Audit trail (CloudTrail)
- ✅ Encryption at rest (KMS)
- ✅ Versioning support

**Cons**:
- ❌ Higher cost ($0.40/secret/month + $0.05/10K API calls)
- ❌ Rotation requires Lambda functions
- ❌ Limited to AWS services
- ❌ No free tier

### Cost
- **Storage**: $0.40 per secret per month
- **API Calls**: $0.05 per 10,000 calls
- **Rotation**: Lambda execution costs

### Use Cases
- RDS database passwords with auto-rotation
- API keys that rotate monthly
- Third-party service credentials
- Application secrets with audit requirements

### Secrets Manager vs Systems Manager Parameter Store

| Feature | Secrets Manager | Parameter Store |
|---------|----------------|-----------------|
| **Cost** | $0.40/secret/month | Free (Standard), $0.05/parameter/month (Advanced) |
| **Rotation** | Built-in | Manual |
| **Size Limit** | 64 KB | 4 KB (Standard), 8 KB (Advanced) |
| **Encryption** | KMS (required) | Optional KMS |
| **Audit** | CloudTrail | CloudTrail |
| **Use Case** | Secrets needing rotation | Configuration values, simple secrets |

**Decision**: 
- Use **Secrets Manager** for secrets requiring rotation (DB passwords, API keys)
- Use **Parameter Store** for configuration values and simple secrets without rotation

---

## 4. KMS (Key Management Service)

### What It Is
Managed service for creating and controlling encryption keys.

### When to Use
- ✅ Encrypting data at rest (S3, EBS, RDS, etc.)
- ✅ Encrypting data in transit
- ✅ Managing encryption keys
- ✅ Key rotation
- ✅ Cross-account key sharing
- ✅ Compliance requirements (encryption)

### Trade-offs

**Pros**:
- ✅ Fully managed
- ✅ Integrated with AWS services
- ✅ Automatic key rotation
- ✅ Audit trail
- ✅ FIPS 140-2 Level 2 validated
- ✅ Granular permissions

**Cons**:
- ❌ Cost per key and API calls
- ❌ Key deletion has 7-30 day waiting period
- ❌ Regional (keys don't replicate)
- ❌ Not FIPS 140-2 Level 3 (use CloudHSM)

### Cost
- **Customer Managed Keys**: $1.00 per key per month
- **API Calls**: $0.03 per 10,000 requests
- **AWS Managed Keys**: Free (limited control)

### Use Cases
- Encrypting S3 buckets
- Encrypting EBS volumes
- Encrypting RDS databases
- Application-level encryption
- Secrets Manager encryption

### KMS vs CloudHSM

| Feature | KMS | CloudHSM |
|---------|-----|----------|
| **Hardware** | AWS managed | Dedicated HSM |
| **FIPS Level** | Level 2 | Level 3 |
| **Cost** | $1/key/month | $1,500+/month |
| **Multi-tenant** | Yes | No (dedicated) |
| **Performance** | High | Very High |
| **Use Case** | General encryption | High-security, compliance (Level 3) |

**Decision**:
- Use **KMS** for general encryption needs (99% of cases)
- Use **CloudHSM** for FIPS 140-2 Level 3 compliance, dedicated hardware requirements

---

## 5. CloudHSM

### What It Is
Dedicated hardware security module (HSM) for managing encryption keys.

### When to Use
- ✅ FIPS 140-2 Level 3 compliance required
- ✅ Dedicated hardware requirement
- ✅ High-performance cryptographic operations
- ✅ Regulatory compliance (banking, healthcare)
- ✅ Custom key management software

### Trade-offs

**Pros**:
- ✅ FIPS 140-2 Level 3 validated
- ✅ Dedicated hardware (single-tenant)
- ✅ High performance
- ✅ Full control over keys
- ✅ Supports custom applications

**Cons**:
- ❌ Very expensive ($1,500+/month)
- ❌ Requires cluster setup (minimum 2 HSMs)
- ❌ Complex setup and management
- ❌ Not integrated with all AWS services
- ❌ Requires specialized knowledge

### Cost
- **HSM Instance**: ~$1,500 per month per HSM
- **Minimum**: 2 HSMs for HA = $3,000/month
- **Data Transfer**: Standard EC2 data transfer rates

### Use Cases
- Banking/financial services (regulatory compliance)
- Healthcare (HIPAA Level 3 requirements)
- Government (FIPS Level 3)
- High-performance crypto operations
- Custom key management applications

**Decision**: Only use CloudHSM when FIPS 140-2 Level 3 compliance is mandatory.

---

## 6. WAF (Web Application Firewall)

### What It Is
Firewall service protecting web applications from common exploits.

### When to Use
- ✅ Protecting web applications from OWASP Top 10
- ✅ Rate limiting
- ✅ IP-based access control
- ✅ Geographic restrictions
- ✅ SQL injection protection
- ✅ XSS protection

### Trade-offs

**Pros**:
- ✅ Managed rules (AWS and Marketplace)
- ✅ Custom rules
- ✅ Real-time metrics
- ✅ Integration with CloudFront, ALB, API Gateway
- ✅ Cost-effective for moderate traffic

**Cons**:
- ❌ Cost scales with requests
- ❌ Rule complexity can impact latency
- ❌ Requires rule tuning
- ❌ Limited to HTTP/HTTPS

### Cost
- **Web ACL**: $5.00 per web ACL per month
- **Requests**: $1.00 per million requests
- **Rule Groups**: Varies by rule group

### Use Cases
- Public-facing web applications
- API protection
- DDoS mitigation (with Shield)
- Compliance (PCI-DSS requirement)
- Bot protection

### WAF vs Security Groups vs NACLs

| Feature | WAF | Security Groups | NACLs |
|---------|-----|----------------|-------|
| **Layer** | Application (Layer 7) | Instance (Layer 3/4) | Subnet (Layer 3/4) |
| **Scope** | HTTP/HTTPS traffic | EC2 instances | Subnets |
| **Rules** | Content-based | Port/IP-based | Port/IP-based |
| **Cost** | Pay per request | Free | Free |
| **Use Case** | Web app protection | Instance firewall | Network firewall |

**Decision**: Use all three - WAF for application layer, Security Groups for instances, NACLs for network layer.

---

## 7. Shield

### What It Is
Managed DDoS protection service for applications running on AWS.

### When to Use
- ✅ DDoS protection for web applications
- ✅ Always-on protection
- ✅ High availability requirements
- ✅ Cost protection from DDoS attacks

### Trade-offs

**Pros**:
- ✅ Always-on detection and mitigation
- ✅ Automatic protection
- ✅ No application changes needed
- ✅ Cost protection (absorbs DDoS costs)
- ✅ 24/7 DDoS response team (Advanced)

**Cons**:
- ❌ Standard: Free but limited
- ❌ Advanced: Expensive ($3,000/month)
- ❌ Standard doesn't cover all attack types
- ❌ Advanced requires 1-year commitment

### Cost
- **Shield Standard**: Free (automatic)
- **Shield Advanced**: $3,000 per month + data transfer costs
- **Cost Protection**: Absorbs scaling costs during attacks

### Use Cases
- Public-facing applications
- High-value targets
- Compliance requirements
- Cost protection from DDoS

### Shield Standard vs Advanced

| Feature | Standard | Advanced |
|---------|----------|----------|
| **Cost** | Free | $3,000/month |
| **Protection** | Common attacks | All known attacks |
| **Response Time** | Automatic | <1 minute |
| **Cost Protection** | No | Yes |
| **DDoS Response Team** | No | 24/7 |
| **WAF Integration** | Basic | Advanced |

**Decision**: 
- Use **Standard** for most applications (free, automatic)
- Use **Advanced** for high-value targets, compliance, or cost protection needs

---

## 8. GuardDuty

### What It Is
Threat detection service that continuously monitors for malicious activity.

### When to Use
- ✅ Continuous security monitoring
- ✅ Threat detection across accounts
- ✅ Anomaly detection
- ✅ Compliance requirements
- ✅ Multi-account security

### Trade-offs

**Pros**:
- ✅ Machine learning-based detection
- ✅ Continuous monitoring
- ✅ No infrastructure to manage
- ✅ Integrates with Security Hub
- ✅ Multi-account support

**Cons**:
- ❌ Cost scales with data analyzed
- ❌ False positives possible
- ❌ Requires tuning
- ❌ CloudTrail, VPC Flow Logs, DNS logs required

### Cost
- **CloudTrail Events**: $0.40 per million events
- **VPC Flow Logs**: $0.50 per GB ingested
- **DNS Logs**: $0.50 per GB ingested
- **S3 Protection**: $0.20 per million S3 requests

### Use Cases
- Continuous security monitoring
- Threat detection
- Compliance (SOC 2, PCI-DSS)
- Multi-account security posture
- Anomaly detection

### GuardDuty vs Inspector vs Macie

| Feature | GuardDuty | Inspector | Macie |
|---------|-----------|-----------|-------|
| **Focus** | Threat detection | Vulnerability assessment | Data protection |
| **Scope** | Network, API, DNS | EC2, ECR, Lambda | S3 |
| **Type** | Continuous | On-demand/Scheduled | Continuous |
| **Cost Model** | Pay per data | Pay per assessment | Pay per GB |
| **Use Case** | Active threats | Security vulnerabilities | Sensitive data |

**Decision**: Use all three for comprehensive security:
- **GuardDuty**: Active threat detection
- **Inspector**: Vulnerability scanning
- **Macie**: Sensitive data protection

---

## 9. Security Hub

### What It Is
Centralized security management service aggregating security findings from multiple AWS services.

### When to Use
- ✅ Centralized security view
- ✅ Compliance monitoring
- ✅ Multi-account security management
- ✅ Security findings aggregation
- ✅ Automated remediation

### Trade-offs

**Pros**:
- ✅ Single pane of glass for security
- ✅ Aggregates findings from multiple services
- ✅ Compliance standards (CIS, PCI-DSS, etc.)
- ✅ Automated remediation
- ✅ Custom insights

**Cons**:
- ❌ Cost per finding
- ❌ Requires other services (GuardDuty, Inspector, etc.)
- ❌ Can generate many findings
- ❌ Requires tuning

### Cost
- **Security Checks**: $0.0010 per security check per account per month
- **Findings**: First 100,000 free, then $0.0003 per finding
- **Custom Insights**: $0.00003 per insight evaluation

### Use Cases
- Centralized security dashboard
- Compliance reporting
- Multi-account security management
- Security operations center (SOC)
- Automated security remediation

### Security Hub vs Individual Services

| Approach | Individual Services | Security Hub |
|----------|---------------------|--------------|
| **View** | Separate dashboards | Centralized |
| **Cost** | Lower | Higher (aggregation) |
| **Management** | Manual | Automated |
| **Compliance** | Manual | Automated |
| **Use Case** | Single service focus | Comprehensive view |

**Decision**: Use Security Hub when you need centralized security management across multiple services and accounts.

---

## 10. Macie

### What It Is
Security service that uses machine learning to discover and protect sensitive data in S3.

### When to Use
- ✅ S3 data discovery
- ✅ PII/PHI detection
- ✅ Compliance (GDPR, HIPAA)
- ✅ Data classification
- ✅ Unauthorized access detection

### Trade-offs

**Pros**:
- ✅ ML-based sensitive data discovery
- ✅ Automatic classification
- ✅ S3 bucket monitoring
- ✅ Compliance support
- ✅ Integration with Security Hub

**Cons**:
- ❌ S3 only (no other storage)
- ❌ Cost scales with data volume
- ❌ Requires S3 access logging
- ❌ Can be expensive for large buckets

### Cost
- **S3 Bucket Analysis**: $0.10 per GB analyzed (first 1GB free)
- **API Calls**: $0.10 per 1,000 API calls
- **Sensitive Data Discovery**: $1.00 per GB analyzed

### Use Cases
- GDPR compliance (PII detection)
- HIPAA compliance (PHI detection)
- PCI-DSS (credit card data)
- Data classification
- Unauthorized access detection

### Macie vs Manual Scanning

| Approach | Manual | Macie |
|----------|--------|-------|
| **Automation** | Manual | Automated |
| **Coverage** | Limited | Comprehensive |
| **Cost** | Time | Pay per GB |
| **Accuracy** | Variable | ML-based |
| **Scale** | Difficult | Handles large scale |

**Decision**: Use Macie for automated, comprehensive S3 data protection and compliance.

---

## 11. Inspector

### What It Is
Automated security assessment service for applications deployed on EC2.

### When to Use
- ✅ Vulnerability scanning
- ✅ EC2 security assessment
- ✅ ECR image scanning
- ✅ Lambda function scanning
- ✅ Compliance reporting

### Trade-offs

**Pros**:
- ✅ Automated vulnerability scanning
- ✅ EC2, ECR, Lambda support
- ✅ Network reachability analysis
- ✅ Integration with Security Hub
- ✅ Compliance reporting

**Cons**:
- ❌ Cost per assessment
- ❌ Requires agents (EC2)
- ❌ Limited to specific services
- ❌ Can generate many findings

### Cost
- **EC2 Assessment**: $0.15 per instance per assessment
- **ECR Assessment**: $0.10 per image per assessment
- **Lambda Assessment**: $0.15 per function per assessment

### Use Cases
- Vulnerability scanning
- Compliance assessments
- Pre-deployment security checks
- Continuous security monitoring
- Security posture assessment

### Inspector vs GuardDuty

| Feature | Inspector | GuardDuty |
|---------|-----------|-----------|
| **Focus** | Vulnerabilities | Threats |
| **Type** | On-demand/Scheduled | Continuous |
| **Scope** | EC2, ECR, Lambda | Network, API, DNS |
| **Cost Model** | Per assessment | Per data volume |
| **Use Case** | Security posture | Active threats |

**Decision**: Use both - Inspector for vulnerability assessment, GuardDuty for threat detection.

---

## 12. Artifact

### What It Is
On-demand access to AWS compliance reports and certifications.

### When to Use
- ✅ Compliance documentation
- ✅ Audit requirements
- ✅ Customer assurance
- ✅ Regulatory compliance
- ✅ Security certifications

### Trade-offs

**Pros**:
- ✅ Free service
- ✅ On-demand access
- ✅ Multiple compliance frameworks
- ✅ Always up-to-date
- ✅ Official AWS documentation

**Cons**:
- ❌ Read-only (no customization)
- ❌ AWS-only (not your infrastructure)
- ❌ May not cover all requirements
- ❌ Requires interpretation

### Cost
- **Free**: No charges

### Use Cases
- Compliance audits
- Customer security questionnaires
- Regulatory submissions
- Security certifications
- Due diligence

**Decision**: Use Artifact for AWS compliance documentation (always free, always available).

---

## Decision Matrix: When to Use Each Service

### Authentication & Access

| Need | Service | Why |
|------|---------|-----|
| AWS resource access | **IAM** | Free, integrated, fine-grained |
| End-user authentication | **Cognito** | Managed, scales, social login |
| Service-to-service auth | **IAM** | Service roles, IRSA |
| Cross-account access | **IAM** | Role assumption, resource sharing |

### Secrets & Keys

| Need | Service | Why |
|------|---------|-----|
| Secrets with rotation | **Secrets Manager** | Auto-rotation, audit trail |
| Simple configuration | **Parameter Store** | Free, sufficient for config |
| Encryption keys | **KMS** | Managed, integrated, cost-effective |
| FIPS Level 3 compliance | **CloudHSM** | Dedicated HSM, Level 3 |
| General encryption | **KMS** | 99% of use cases |

### Protection & Detection

| Need | Service | Why |
|------|---------|-----|
| Web app protection | **WAF** | OWASP Top 10, rate limiting |
| DDoS protection | **Shield** | Always-on, automatic |
| Threat detection | **GuardDuty** | ML-based, continuous |
| Vulnerability scanning | **Inspector** | EC2, ECR, Lambda assessment |
| S3 data protection | **Macie** | ML-based, compliance |

### Management & Compliance

| Need | Service | Why |
|------|---------|-----|
| Centralized security | **Security Hub** | Aggregates findings |
| Compliance docs | **Artifact** | Free, official, on-demand |
| Multi-account security | **Security Hub** | Cross-account aggregation |

---

## Cost Comparison Summary

### Free Services
- ✅ **IAM**: Free
- ✅ **Artifact**: Free
- ✅ **Shield Standard**: Free (automatic)
- ✅ **Parameter Store Standard**: Free

### Low Cost (< $100/month typical)
- 💰 **Cognito**: ~$0.0055 per MAU (first 50K free)
- 💰 **KMS**: $1 per key + $0.03 per 10K calls
- 💰 **WAF**: $5 per web ACL + $1 per million requests
- 💰 **GuardDuty**: ~$0.40-0.50 per million events/GB

### Medium Cost ($100-1000/month typical)
- 💰💰 **Secrets Manager**: $0.40 per secret + API calls
- 💰💰 **Security Hub**: $0.001 per check + findings
- 💰💰 **Macie**: $0.10-1.00 per GB analyzed
- 💰💰 **Inspector**: $0.10-0.15 per assessment

### High Cost (> $1000/month)
- 💰💰💰 **CloudHSM**: ~$1,500+ per HSM (minimum 2)
- 💰💰💰 **Shield Advanced**: $3,000 per month

---

## Best Practices: Service Combinations

### Comprehensive Security Stack

```
┌─────────────────────────────────────────────────┐
│           Security Hub (Centralized)              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │GuardDuty │  │Inspector │  │  Macie  │       │
│  └──────────┘  └──────────┘  └──────────┘       │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│              Application Layer                   │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │   WAF    │  │  Shield  │  │ Cognito  │      │
│  └──────────┘  └──────────┘  └──────────┘      │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│              Infrastructure Layer                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │   IAM    │  │   KMS    │  │ Secrets  │      │
│  └──────────┘  └──────────┘  └──────────┘      │
└─────────────────────────────────────────────────┘
```

### Recommended Combinations

**1. Basic Security (Cost-Conscious)**
- IAM (free)
- KMS (encryption)
- Shield Standard (free)
- Parameter Store (free)

**2. Standard Security**
- IAM + Cognito (authentication)
- KMS (encryption)
- Secrets Manager (secrets)
- WAF + Shield Standard (protection)
- GuardDuty (threat detection)

**3. Enterprise Security**
- All services above +
- Security Hub (centralized)
- Inspector (vulnerabilities)
- Macie (data protection)
- Shield Advanced (if needed)
- CloudHSM (if Level 3 required)

---

## Real-World Scenarios

### Scenario 1: Startup Web Application

**Requirements**: User authentication, basic security, cost-conscious

**Recommended Stack**:
- **Cognito**: User authentication ($0.0055/MAU)
- **IAM**: AWS resource access (free)
- **KMS**: Encryption (S3, RDS) (~$1-5/month)
- **Parameter Store**: App configuration (free)
- **WAF**: Basic protection (~$10-50/month)
- **Shield Standard**: DDoS protection (free)

**Monthly Cost**: ~$20-60

---

### Scenario 2: Enterprise E-Commerce Platform

**Requirements**: High security, compliance, comprehensive protection

**Recommended Stack**:
- **Cognito**: User authentication
- **IAM**: Access management
- **KMS**: Encryption
- **Secrets Manager**: Database credentials with rotation
- **WAF**: Advanced protection
- **Shield Advanced**: DDoS protection
- **GuardDuty**: Threat detection
- **Security Hub**: Centralized management
- **Inspector**: Vulnerability scanning
- **Macie**: S3 data protection

**Monthly Cost**: ~$5,000-10,000+

---

### Scenario 3: Financial Services (Compliance)

**Requirements**: FIPS Level 3, regulatory compliance, audit trail

**Recommended Stack**:
- **IAM**: Access management
- **CloudHSM**: FIPS Level 3 keys ($3,000+/month)
- **KMS**: General encryption
- **Secrets Manager**: Credentials with audit
- **GuardDuty**: Threat detection
- **Security Hub**: Compliance reporting
- **Inspector**: Regular assessments
- **Artifact**: Compliance documentation

**Monthly Cost**: ~$4,000-8,000+

---

## Summary: Key Trade-offs

### Cost vs Features

| Service | Cost | Features | Best For |
|---------|------|----------|----------|
| **IAM** | Free | Basic access control | Everyone |
| **Cognito** | Low | User management | Customer apps |
| **KMS** | Low | Encryption | Everyone |
| **CloudHSM** | Very High | Level 3 compliance | Regulated industries |
| **Secrets Manager** | Medium | Rotation | Secrets needing rotation |
| **WAF** | Low-Medium | Web protection | Public web apps |
| **Shield Advanced** | High | DDoS protection | High-value targets |
| **GuardDuty** | Low-Medium | Threat detection | Continuous monitoring |
| **Security Hub** | Medium | Centralized view | Multi-service management |
| **Macie** | Medium-High | Data protection | S3 compliance |
| **Inspector** | Low-Medium | Vulnerability scanning | Security assessments |

### Decision Framework

1. **Start with Free**: IAM, Shield Standard, Parameter Store
2. **Add Essentials**: KMS, Cognito (if needed), WAF
3. **Add Detection**: GuardDuty, Inspector
4. **Add Management**: Security Hub (if using multiple services)
5. **Add Advanced**: Shield Advanced, Macie, CloudHSM (if required)

### Cost Optimization Tips

- ✅ Use Parameter Store for simple secrets (free)
- ✅ Use Secrets Manager only when rotation needed
- ✅ Start with Shield Standard (free)
- ✅ Use KMS instead of CloudHSM unless Level 3 required
- ✅ Monitor GuardDuty costs (can scale with traffic)
- ✅ Use Security Hub only if managing multiple services
- ✅ Schedule Inspector assessments (not continuous)

---

*This trade-off analysis helps you make informed decisions about which AWS Security & Identity services to use based on your specific requirements, budget, and compliance needs.*

