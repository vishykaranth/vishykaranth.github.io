# System Design Interview Tips - Part 4: Handle Failures & Security

## Overview

This document provides in-depth guidance on handling failures and implementing security in system design interviews:
1. **Handle Failures**: Discuss failure scenarios and recovery
2. **Security**: Authentication, authorization, encryption

---

## 1. Handle Failures: Discuss Failure Scenarios and Recovery

### 1.1 Why Failure Handling is Critical

**Reality**: Systems fail. Hardware fails, networks fail, software has bugs.

**Statistics**:
- Hard drives: 2-4% annual failure rate
- Network: 0.1-1% packet loss
- Servers: 1-5% annual failure rate
- Data centers: Regional outages occur

**Interview Importance**:
- Shows production experience
- Demonstrates system thinking
- Critical for high-availability systems
- Differentiates senior engineers

### 1.2 Types of Failures

#### A. Single Point of Failure (SPOF)

**Definition**: A component whose failure causes entire system failure

**Common SPOFs**:
- Single database server
- Single load balancer
- Single message queue
- Single cache server
- Single network link

**Example: Single Database**

```
❌ Bad Design:
Client → App Server → Single Database
                      ↓
                  (SPOF - if fails, system down)

✅ Good Design:
Client → App Server → Master DB
                      ↓
                   Replica 1
                   Replica 2
                   Replica 3
                  (No SPOF - replicas can take over)
```

#### B. Cascading Failures

**Definition**: Failure in one component causes failures in others

**Example: Cache Stampede**

```
Scenario:
1. Cache expires for popular key
2. All requests miss cache simultaneously
3. All requests hit database
4. Database overloads
5. Database fails
6. Entire system fails

Prevention:
- Cache warming
- Staggered expiration
- Rate limiting
- Circuit breakers
```

#### C. Partial Failures

**Definition**: Some components fail while others continue working

**Example: Database Replica Failure**

```
Scenario:
- Master DB: Working
- Replica 1: Working
- Replica 2: Failed
- Replica 3: Working

Impact:
- Reads can continue (other replicas)
- Writes can continue (master)
- Reduced read capacity
- Need to repair replica
```

### 1.3 Failure Scenarios by Component

#### A. Database Failures

**Failure Types**:
1. **Complete Failure**: Database server crashes
2. **Network Partition**: Database unreachable
3. **Disk Failure**: Data corruption or loss
4. **Performance Degradation**: Slow queries

**Recovery Strategies**:

```
1. REPLICATION
Master-Slave:
- Master handles writes
- Slaves handle reads
- If master fails: Promote slave to master

Master-Master:
- Both handle reads/writes
- Automatic failover
- Conflict resolution needed

2. BACKUP AND RESTORE
- Regular backups (daily/hourly)
- Point-in-time recovery
- Test restore procedures

3. MONITORING
- Health checks
- Query performance monitoring
- Disk space monitoring
- Alert on anomalies

4. CIRCUIT BREAKER
- Stop sending requests if DB is down
- Fail fast
- Retry with backoff
```

**Example: Database Failure Handling**

```
Design:
┌─────────┐
│ Client  │
└────┬────┘
     │
┌────▼────────┐
│ App Server  │
└────┬────────┘
     │
     ├───→ Master DB (writes)
     │
     ├───→ Replica 1 (reads)
     ├───→ Replica 2 (reads)
     └───→ Replica 3 (reads)

Failure Scenarios:

1. Master DB fails:
   - Automatic failover to replica
   - Promote replica to master
   - Update DNS/config
   - Recovery time: < 1 minute

2. Replica fails:
   - Remove from read pool
   - Continue with other replicas
   - Repair replica asynchronously
   - Recovery time: < 30 seconds

3. All databases fail:
   - Serve from cache (stale data)
   - Show maintenance message
   - Restore from backup
   - Recovery time: < 1 hour
```

#### B. Cache Failures

**Failure Types**:
1. **Cache Server Down**: Redis/Memcached unavailable
2. **Cache Eviction**: Memory full, data evicted
3. **Network Issues**: Cache unreachable
4. **Data Corruption**: Invalid data in cache

**Recovery Strategies**:

```
1. CACHE-ASIDE PATTERN
- Application checks cache
- If miss: Query database
- If cache down: Query database directly
- Graceful degradation

2. MULTI-LEVEL CACHING
- L1: Application cache
- L2: Distributed cache
- L3: Database
- If L2 fails, use L1 and L3

3. CACHE WARMING
- Pre-populate cache on startup
- Background refresh
- Staggered expiration

4. CIRCUIT BREAKER
- If cache fails repeatedly, bypass it
- Direct to database
- Retry cache later
```

**Example: Cache Failure Handling**

```
Normal Flow:
Request → App Cache (L1) → Redis (L2) → Database (L3)

Cache Failure Flow:
Request → App Cache (L1) → Redis (L2) [FAILED]
                              ↓
                         Database (L3)
                         (Slightly slower, but works)

Multi-Cache Failure:
Request → App Cache (L1) [FAILED] → Redis (L2) [FAILED]
                                      ↓
                                 Database (L3)
                                 (Slower, but functional)
```

#### C. Application Server Failures

**Failure Types**:
1. **Server Crash**: Process dies
2. **Out of Memory**: OOM killer
3. **CPU Exhaustion**: 100% CPU
4. **Network Issues**: Server unreachable

**Recovery Strategies**:

```
1. LOAD BALANCER HEALTH CHECKS
- Regular health checks
- Remove unhealthy servers
- Automatic recovery

2. AUTO-SCALING
- Scale up on high load
- Scale down on low load
- Replace failed instances

3. GRACEFUL SHUTDOWN
- Stop accepting new requests
- Finish in-flight requests
- Clean shutdown

4. MONITORING
- CPU, memory, disk monitoring
- Application metrics
- Alert on thresholds
```

**Example: Application Server Failure**

```
Design:
┌─────────┐
│ Client  │
└────┬────┘
     │
┌────▼────────┐
│Load Balancer│
└────┬────────┘
     │
     ├───→ Server 1 [HEALTHY]
     ├───→ Server 2 [FAILED] ← Removed from pool
     ├───→ Server 3 [HEALTHY]
     └───→ Server 4 [HEALTHY]

Recovery:
1. Health check detects failure
2. Remove server from pool
3. Auto-scaling launches new server
4. New server joins pool
5. Recovery time: 2-5 minutes
```

#### D. Network Failures

**Failure Types**:
1. **Network Partition**: Split-brain
2. **High Latency**: Slow network
3. **Packet Loss**: Data loss
4. **DNS Failure**: Can't resolve domains

**Recovery Strategies**:

```
1. RETRY MECHANISMS
- Exponential backoff
- Jitter
- Max retries

2. TIMEOUTS
- Connection timeout
- Read timeout
- Write timeout

3. CIRCUIT BREAKER
- Stop sending requests if failures
- Fail fast
- Retry after cooldown

4. MULTIPLE ENDPOINTS
- Primary and secondary
- Automatic failover
- Geographic distribution
```

### 1.4 Failure Handling Patterns

#### Pattern 1: Retry with Exponential Backoff

```
Attempt 1: Immediate
Attempt 2: Wait 1 second
Attempt 3: Wait 2 seconds
Attempt 4: Wait 4 seconds
Attempt 5: Wait 8 seconds
Max attempts: 5

Formula: wait_time = min(2^attempt, max_wait)
```

**Example Implementation**:

```java
public class RetryWithBackoff {
    private static final int MAX_ATTEMPTS = 5;
    private static final long MAX_WAIT = 10000; // 10 seconds
    
    public void executeWithRetry(Runnable operation) {
        for (int attempt = 0; attempt < MAX_ATTEMPTS; attempt++) {
            try {
                operation.run();
                return; // Success
            } catch (Exception e) {
                if (attempt == MAX_ATTEMPTS - 1) {
                    throw e; // Last attempt failed
                }
                long waitTime = Math.min(
                    (long) Math.pow(2, attempt) * 1000,
                    MAX_WAIT
                );
                Thread.sleep(waitTime);
            }
        }
    }
}
```

#### Pattern 2: Circuit Breaker

```
States:
1. CLOSED: Normal operation
2. OPEN: Too many failures, fail fast
3. HALF_OPEN: Testing if service recovered

Transitions:
CLOSED → OPEN: Failure threshold reached
OPEN → HALF_OPEN: After timeout
HALF_OPEN → CLOSED: Success
HALF_OPEN → OPEN: Still failing
```

**Example Implementation**:

```java
public class CircuitBreaker {
    private State state = State.CLOSED;
    private int failureCount = 0;
    private long lastFailureTime = 0;
    private static final int FAILURE_THRESHOLD = 5;
    private static final long TIMEOUT = 60000; // 1 minute
    
    public <T> T execute(Supplier<T> operation) {
        if (state == State.OPEN) {
            if (System.currentTimeMillis() - lastFailureTime > TIMEOUT) {
                state = State.HALF_OPEN;
            } else {
                throw new CircuitBreakerOpenException();
            }
        }
        
        try {
            T result = operation.get();
            if (state == State.HALF_OPEN) {
                state = State.CLOSED;
                failureCount = 0;
            }
            return result;
        } catch (Exception e) {
            failureCount++;
            lastFailureTime = System.currentTimeMillis();
            if (failureCount >= FAILURE_THRESHOLD) {
                state = State.OPEN;
            }
            throw e;
        }
    }
}
```

#### Pattern 3: Bulkhead

**Definition**: Isolate failures to prevent cascading

**Example: Thread Pool Isolation**

```
Instead of:
Single thread pool for all requests
→ One slow request blocks all

Use:
Separate thread pools:
- Fast requests: 100 threads
- Slow requests: 10 threads
- Background jobs: 5 threads

→ Slow requests don't block fast requests
```

#### Pattern 4: Timeout

**Definition**: Fail fast if operation takes too long

**Example: Database Query Timeout**

```
Without timeout:
Query hangs → All threads blocked → System unresponsive

With timeout:
Query timeout: 5 seconds
→ Fail fast → System remains responsive
```

### 1.5 Disaster Recovery

#### A. Backup Strategies

```
1. FULL BACKUP
- Complete system backup
- Daily/weekly
- Slow, but complete

2. INCREMENTAL BACKUP
- Only changed data
- Faster
- Need full backup + increments

3. CONTINUOUS BACKUP
- Real-time replication
- Point-in-time recovery
- More expensive
```

#### B. Recovery Objectives

```
RPO (Recovery Point Objective):
- Maximum acceptable data loss
- Example: 1 hour (can lose 1 hour of data)

RTO (Recovery Time Objective):
- Maximum acceptable downtime
- Example: 4 hours (system back in 4 hours)
```

#### C. Disaster Recovery Plan

```
1. BACKUP
   - Regular backups
   - Test restore procedures
   - Off-site backups

2. REPLICATION
   - Multi-region replication
   - Automatic failover
   - Data synchronization

3. MONITORING
   - Health checks
   - Alerting
   - Runbooks

4. TESTING
   - Regular DR drills
   - Failover testing
   - Recovery testing
```

### 1.6 Interview Tips

**Tip 1: Think About All Failure Points**
```
✅ "Let me think about failure scenarios:
   - Database failure
   - Cache failure
   - Network failure
   - Application server failure"

❌ "The system should handle failures"
```

**Tip 2: Discuss Recovery Strategies**
```
✅ "If the database fails, we'll:
   1. Failover to replica (< 1 minute)
   2. Serve from cache (stale data)
   3. Show maintenance message"

❌ "We'll handle database failures"
```

**Tip 3: Quantify Recovery Times**
```
✅ "RTO: 5 minutes, RPO: 1 hour"

❌ "We'll recover quickly"
```

**Tip 4: Mention Monitoring**
```
✅ "We'll monitor:
   - Health checks every 10 seconds
   - Alert on failures
   - Automatic recovery"

❌ "We'll monitor the system"
```

---

## 2. Security: Authentication, Authorization, Encryption

### 2.1 Why Security Matters

**Threats**:
- Unauthorized access
- Data breaches
- DDoS attacks
- Injection attacks
- Man-in-the-middle attacks

**Impact**:
- Financial loss
- Reputation damage
- Legal liability
- User trust

### 2.2 Security Layers

```
┌─────────────────────────────────────────┐
│         SECURITY LAYERS                 │
├─────────────────────────────────────────┤
│ 1. Network Security                     │
│    - Firewall                           │
│    - DDoS protection                    │
│    - VPN                                │
├─────────────────────────────────────────┤
│ 2. Application Security                 │
│    - Authentication                    │
│    - Authorization                     │
│    - Input validation                  │
├─────────────────────────────────────────┤
│ 3. Data Security                        │
│    - Encryption at rest                 │
│    - Encryption in transit             │
│    - Data masking                       │
├─────────────────────────────────────────┤
│ 4. Infrastructure Security              │
│    - Access control                    │
│    - Audit logging                     │
│    - Security monitoring               │
└─────────────────────────────────────────┘
```

### 2.3 Authentication

#### A. Authentication Methods

**1. Password-Based Authentication**

```
Flow:
1. User provides username/password
2. Server hashes password
3. Compare with stored hash
4. Issue token if match

Security:
- Hash passwords (bcrypt, Argon2)
- Salt passwords
- Rate limiting
- Account lockout
```

**2. Token-Based Authentication (JWT)**

```
Flow:
1. User authenticates with credentials
2. Server issues JWT token
3. Client stores token
4. Client sends token with requests
5. Server validates token

JWT Structure:
Header.Payload.Signature

Example:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

**3. OAuth 2.0**

```
Flow:
1. User clicks "Login with Google"
2. Redirect to Google
3. User authorizes
4. Google redirects with code
5. Exchange code for token
6. Use token for API calls

Benefits:
- No password storage
- Third-party authentication
- Standard protocol
```

**4. Multi-Factor Authentication (MFA)**

```
Factors:
1. Something you know (password)
2. Something you have (phone, token)
3. Something you are (biometric)

Example:
1. User enters password
2. System sends OTP to phone
3. User enters OTP
4. Access granted
```

#### B. Session Management

```
Stateless Sessions (JWT):
- Token contains all info
- No server-side storage
- Scalable
- Can't revoke easily

Stateful Sessions (Server-side):
- Session ID in cookie
- Session data in server/cache
- Can revoke easily
- Requires session storage
```

**Example: Session Management**

```
Option 1: JWT (Stateless)
Client → Server (with JWT)
Server validates JWT signature
→ No database lookup needed
→ Scalable

Option 2: Server-Side Session
Client → Server (with session ID)
Server looks up session in Redis
→ Can revoke sessions
→ Requires Redis
```

### 2.4 Authorization

#### A. Authorization Models

**1. Role-Based Access Control (RBAC)**

```
Roles:
- Admin: Full access
- User: Limited access
- Guest: Read-only

Example:
if (user.role == "admin") {
    // Allow delete
} else {
    // Deny delete
}
```

**2. Attribute-Based Access Control (ABAC)**

```
Attributes:
- User department
- Resource sensitivity
- Time of day
- Location

Example:
if (user.department == "finance" && 
    resource.sensitivity == "financial" &&
    time.isBusinessHours()) {
    // Allow access
}
```

**3. Resource-Based Authorization**

```
Check ownership:
if (resource.ownerId == user.id) {
    // Allow access
} else {
    // Deny access
}
```

#### B. Permission Model

```
Permissions:
- Read
- Write
- Delete
- Admin

Example:
User → Roles → Permissions → Resources

User "Alice" has role "Editor"
Role "Editor" has permissions: [Read, Write]
Permissions apply to resources: [Posts, Comments]
```

### 2.5 Encryption

#### A. Encryption at Rest

**Definition**: Encrypt data stored on disk

**Methods**:
1. **Database Encryption**: TDE (Transparent Data Encryption)
2. **File System Encryption**: Encrypted volumes
3. **Application-Level Encryption**: Encrypt before storing

**Example: Database Encryption**

```
Without encryption:
Database file: user_data.db
→ Anyone with file access can read data

With encryption:
Database file: user_data.db.encrypted
→ Requires key to decrypt
→ Key stored in key management service
```

#### B. Encryption in Transit

**Definition**: Encrypt data during transmission

**Methods**:
1. **TLS/SSL**: HTTPS, WSS
2. **VPN**: Encrypted tunnel
3. **Application-Level**: Encrypt payload

**Example: HTTPS**

```
Without HTTPS:
Client → Server (plaintext)
→ Anyone on network can intercept

With HTTPS:
Client → Server (encrypted with TLS)
→ Only client and server can read
```

#### C. Key Management

```
Key Storage:
- Hardware Security Module (HSM)
- Cloud Key Management (AWS KMS, Azure Key Vault)
- Secrets Manager

Key Rotation:
- Regular rotation (every 90 days)
- Multiple key versions
- Gradual migration
```

### 2.6 Security Best Practices

#### A. Input Validation

```
❌ Bad:
String query = "SELECT * FROM users WHERE id = " + userId;
// SQL injection vulnerability

✅ Good:
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE id = ?"
);
stmt.setInt(1, userId);
// Parameterized query, safe
```

#### B. Output Encoding

```
❌ Bad:
response.write("<div>" + userInput + "</div>");
// XSS vulnerability

✅ Good:
response.write("<div>" + escapeHtml(userInput) + "</div>");
// HTML escaped, safe
```

#### C. Rate Limiting

```
Prevent abuse:
- Login attempts: 5 per minute
- API calls: 1000 per hour
- Password reset: 3 per day

Implementation:
- Token bucket algorithm
- Sliding window
- Distributed rate limiting (Redis)
```

#### D. Security Headers

```
HTTP Security Headers:
- Content-Security-Policy
- X-Frame-Options
- X-Content-Type-Options
- Strict-Transport-Security
- X-XSS-Protection
```

### 2.7 Security Architecture Example

**Design: Secure API System**

```
┌─────────────────────────────────────────┐
│              CLIENT                      │
└──────┬──────────────────────────────────┘
       │ HTTPS
       │
┌──────▼──────────────────────────────────┐
│         API GATEWAY                      │
│  - SSL/TLS termination                   │
│  - Rate limiting                         │
│  - Request validation                    │
└──────┬──────────────────────────────────┘
       │
┌──────▼──────────────────────────────────┐
│      AUTHENTICATION SERVICE              │
│  - JWT validation                       │
│  - Token refresh                        │
└──────┬──────────────────────────────────┘
       │
┌──────▼──────────────────────────────────┐
│      AUTHORIZATION SERVICE              │
│  - Permission check                     │
│  - Role validation                      │
└──────┬──────────────────────────────────┘
       │
┌──────▼──────────────────────────────────┐
│      APPLICATION SERVICE                │
│  - Input validation                     │
│  - Business logic                       │
└──────┬──────────────────────────────────┘
       │
┌──────▼──────────────────────────────────┐
│         DATABASE                        │
│  - Encrypted at rest                    │
│  - Encrypted connections                │
└─────────────────────────────────────────┘
```

### 2.8 Interview Tips

**Tip 1: Cover All Security Aspects**
```
✅ "For security, we need:
   - Authentication (JWT)
   - Authorization (RBAC)
   - Encryption (TLS, at rest)
   - Input validation
   - Rate limiting"

❌ "We'll use HTTPS"
```

**Tip 2: Explain Security Choices**
```
✅ "We're using JWT for stateless authentication
   because it scales better than server-side sessions"

❌ "We'll use JWT"
```

**Tip 3: Mention Compliance**
```
✅ "For healthcare data, we need HIPAA compliance:
   - Encryption at rest and in transit
   - Access logging
   - Audit trails"

❌ "We'll secure the data"
```

**Tip 4: Discuss Threat Model**
```
✅ "Main threats:
   - Unauthorized access → Authentication
   - Data breaches → Encryption
   - DDoS → Rate limiting, CDN
   - Injection attacks → Input validation"

❌ "We'll secure against attacks"
```

---

## Summary

### Handle Failures
1. ✅ Identify all failure points
2. ✅ Design for redundancy
3. ✅ Implement retry mechanisms
4. ✅ Use circuit breakers
5. ✅ Plan disaster recovery

### Security
1. ✅ Implement authentication
2. ✅ Enforce authorization
3. ✅ Encrypt data (at rest and in transit)
4. ✅ Validate inputs
5. ✅ Monitor and audit

**Key Takeaway**: Failure handling and security are not optional. They're essential for production systems and demonstrate senior-level thinking.

