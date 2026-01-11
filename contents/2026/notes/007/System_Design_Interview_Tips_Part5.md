# System Design Interview Tips - Part 5: Monitoring & Iterate

## Overview

This document provides in-depth guidance on monitoring and iterative refinement in system design interviews:
1. **Monitoring**: Metrics, logging, alerting
2. **Iterate**: Refine design based on feedback

---

## 1. Monitoring: Metrics, Logging, Alerting

### 1.1 Why Monitoring is Critical

**Reality**: You can't improve what you don't measure.

**Benefits**:
- Detect issues before users notice
- Understand system behavior
- Make data-driven decisions
- Debug problems faster
- Optimize performance

**Interview Importance**:
- Shows production experience
- Demonstrates operational thinking
- Critical for reliability
- Differentiates senior engineers

### 1.2 Three Pillars of Observability

```
┌─────────────────────────────────────────┐
│      THREE PILLARS OF OBSERVABILITY     │
├─────────────────────────────────────────┤
│                                         │
│  1. METRICS                            │
│     - Quantitative measurements         │
│     - Aggregated over time             │
│     - Examples: CPU, memory, RPS       │
│                                         │
│  2. LOGS                                │
│     - Event records                     │
│     - Timestamped                      │
│     - Examples: Errors, requests        │
│                                         │
│  3. TRACES                              │
│     - Request flow across services      │
│     - Distributed tracing              │
│     - Examples: Request latency        │
│                                         │
└─────────────────────────────────────────┘
```

### 1.3 Metrics

#### A. Types of Metrics

**1. System Metrics**

```
CPU Usage:
- Current CPU utilization
- CPU per core
- Load average

Memory Usage:
- Total memory
- Used memory
- Available memory
- Cache/buffer memory

Disk Usage:
- Disk space used
- Disk I/O (read/write)
- Disk latency

Network:
- Bandwidth (in/out)
- Packet loss
- Network latency
```

**2. Application Metrics**

```
Request Rate:
- Requests per second (RPS)
- Requests per minute
- Peak vs average

Latency:
- P50 (median)
- P95
- P99
- P99.9

Error Rate:
- Error percentage
- Error count
- Error types

Throughput:
- Successful requests/sec
- Failed requests/sec
- Total requests/sec
```

**3. Business Metrics**

```
User Metrics:
- Active users (DAU, MAU)
- New users
- User retention

Revenue Metrics:
- Revenue per user
- Conversion rate
- Transaction volume

Engagement Metrics:
- Time spent
- Actions per user
- Feature usage
```

#### B. Metric Collection

**Push vs Pull Model**

```
Push Model (StatsD):
Application → StatsD → Time-series DB
- Application sends metrics
- Lower overhead
- Real-time

Pull Model (Prometheus):
Prometheus → Application (scrapes)
- Prometheus pulls metrics
- Centralized
- Standard format
```

**Example: Metric Collection Architecture**

```
┌─────────────────────────────────────────┐
│         APPLICATION SERVICES            │
│  ┌──────────┐  ┌──────────┐            │
│  │ Service1 │  │ Service2 │            │
│  └────┬─────┘  └────┬─────┘            │
│       │             │                   │
│       └──────┬──────┘                   │
│              │                          │
│       ┌──────▼──────┐                   │
│       │   StatsD    │                   │
│       └──────┬──────┘                   │
└──────────────┼──────────────────────────┘
               │
       ┌───────▼───────┐
       │  Prometheus  │
       │ (Time-series)│
       └───────┬───────┘
               │
       ┌───────▼───────┐
       │   Grafana     │
       │ (Visualization)│
       └───────────────┘
```

#### C. Key Metrics to Monitor

**Golden Signals (Google SRE)**

```
1. LATENCY
   - Request latency
   - P50, P95, P99
   - Error latency

2. TRAFFIC
   - Requests per second
   - Concurrent users
   - Data transfer

3. ERRORS
   - Error rate
   - Error types
   - 4xx vs 5xx errors

4. SATURATION
   - Resource utilization
   - CPU, memory, disk
   - Queue depth
```

**Example: Key Metrics Dashboard**

```
┌─────────────────────────────────────────┐
│         SYSTEM DASHBOARD                │
├─────────────────────────────────────────┤
│                                         │
│  Requests/sec: 10,000                   │
│  Latency P99: 200ms                     │
│  Error Rate: 0.1%                       │
│  CPU Usage: 60%                         │
│  Memory Usage: 70%                      │
│  Disk I/O: 500 IOPS                     │
│                                         │
└─────────────────────────────────────────┘
```

### 1.4 Logging

#### A. Log Levels

```
DEBUG: Detailed information for debugging
INFO: General information about operations
WARN: Warning messages (potential issues)
ERROR: Error messages (operations failed)
FATAL: Critical errors (system may stop)
```

**Example: Log Levels**

```
DEBUG: "Processing user request: userId=123, action=login"
INFO: "User logged in: userId=123"
WARN: "High memory usage: 85%"
ERROR: "Database connection failed: timeout after 5s"
FATAL: "Out of memory: system shutting down"
```

#### B. Structured Logging

```
❌ Unstructured:
"User 123 logged in at 10:30"

✅ Structured (JSON):
{
  "timestamp": "2024-01-15T10:30:00Z",
  "level": "INFO",
  "service": "auth-service",
  "userId": "123",
  "action": "login",
  "ip": "192.168.1.1",
  "userAgent": "Mozilla/5.0..."
}
```

**Benefits**:
- Easy to parse
- Searchable
- Queryable
- Machine-readable

#### C. Log Aggregation

**Architecture**:

```
┌─────────────────────────────────────────┐
│         APPLICATION SERVICES            │
│  ┌──────────┐  ┌──────────┐            │
│  │ Service1 │  │ Service2 │            │
│  └────┬─────┘  └────┬─────┘            │
│       │             │                   │
│       └──────┬──────┘                   │
│              │                          │
│       ┌──────▼──────┐                   │
│       │  Log Agent  │                   │
│       │  (Filebeat) │                   │
│       └──────┬──────┘                   │
└──────────────┼──────────────────────────┘
               │
       ┌───────▼───────┐
       │  Log Aggregator│
       │  (Logstash)   │
       └───────┬───────┘
               │
       ┌───────▼───────┐
       │  Log Storage  │
       │  (Elasticsearch)│
       └───────┬───────┘
               │
       ┌───────▼───────┐
       │  Visualization│
       │  (Kibana)     │
       └───────────────┘
```

#### D. What to Log

```
1. REQUEST LOGS
   - Request ID
   - User ID
   - Endpoint
   - Method
   - Status code
   - Latency
   - Request/response size

2. ERROR LOGS
   - Error message
   - Stack trace
   - Context (user, request)
   - Timestamp

3. AUDIT LOGS
   - User actions
   - Data changes
   - Access attempts
   - Security events

4. PERFORMANCE LOGS
   - Slow queries
   - Cache misses
   - External API calls
   - Database operations
```

### 1.5 Alerting

#### A. Alert Types

**1. Threshold Alerts**

```
CPU usage > 80% for 5 minutes
→ Alert: "High CPU usage"

Error rate > 1% for 2 minutes
→ Alert: "High error rate"

Latency P99 > 500ms for 3 minutes
→ Alert: "High latency"
```

**2. Anomaly Alerts**

```
Unusual traffic pattern detected
→ Alert: "Traffic spike"

Unusual error pattern detected
→ Alert: "Error anomaly"
```

**3. Composite Alerts**

```
CPU > 80% AND Memory > 80% AND Error rate > 1%
→ Alert: "System under stress"
```

#### B. Alert Severity

```
CRITICAL: System down, immediate action needed
HIGH: Major issue, action within 1 hour
MEDIUM: Issue detected, action within 4 hours
LOW: Minor issue, action within 24 hours
INFO: Informational, no action needed
```

#### C. Alerting Best Practices

```
1. AVOID ALERT FATIGUE
   - Only alert on actionable issues
   - Use appropriate thresholds
   - Group related alerts

2. ALERT ON SYMPTOMS, NOT CAUSES
   - Alert: "High error rate"
   - Not: "Database connection pool exhausted"
   - (Database issue is cause, error rate is symptom)

3. SET APPROPRIATE THRESHOLDS
   - Too low: False positives
   - Too high: Miss real issues
   - Use P95/P99, not averages

4. INCLUDE CONTEXT
   - What happened
   - When it happened
   - Where it happened
   - Impact
   - Suggested actions
```

**Example: Alert Configuration**

```
Alerts:
1. Error Rate > 1% for 2 minutes
   Severity: HIGH
   Notification: PagerDuty, Slack

2. Latency P99 > 500ms for 5 minutes
   Severity: MEDIUM
   Notification: Email, Slack

3. CPU > 90% for 10 minutes
   Severity: HIGH
   Notification: PagerDuty

4. Disk space < 10%
   Severity: CRITICAL
   Notification: PagerDuty, Phone
```

### 1.6 Distributed Tracing

#### A. What is Distributed Tracing

**Definition**: Track request flow across multiple services

**Example: E-commerce Request**

```
Request: GET /api/orders/123

Trace:
┌─────────────────────────────────────────┐
│ API Gateway (10ms)                      │
│   ├─> Auth Service (5ms)                │
│   └─> Order Service (50ms)              │
│       ├─> Database (30ms)               │
│       └─> Payment Service (20ms)        │
│           └─> External API (15ms)      │
└─────────────────────────────────────────┘

Total: 10 + 5 + 50 = 65ms
```

#### B. Tracing Tools

```
1. Jaeger
   - Open-source
   - UI for visualization
   - Supports multiple languages

2. Zipkin
   - Open-source
   - Simple setup
   - Good for microservices

3. AWS X-Ray
   - Managed service
   - AWS integration
   - Automatic instrumentation
```

### 1.7 Monitoring Architecture Example

**Complete Monitoring Stack**:

```
┌─────────────────────────────────────────┐
│         APPLICATION SERVICES            │
│  ┌──────────┐  ┌──────────┐            │
│  │ Service1 │  │ Service2 │            │
│  └────┬─────┘  └────┬─────┘            │
│       │             │                   │
│       ├───Metrics───┤                   │
│       ├───Logs──────┤                   │
│       └───Traces────┤                   │
└──────────────┼──────────────────────────┘
               │
       ┌───────▼───────┐
       │  Monitoring   │
       │  Infrastructure│
       │  - Prometheus │
       │  - ELK Stack  │
       │  - Jaeger     │
       └───────┬───────┘
               │
       ┌───────▼───────┐
       │  Alerting     │
       │  - Alertmanager│
       │  - PagerDuty  │
       └───────┬───────┘
               │
       ┌───────▼───────┐
       │  Visualization│
       │  - Grafana    │
       │  - Kibana     │
       └───────────────┘
```

### 1.8 Interview Tips

**Tip 1: Mention Key Metrics**
```
✅ "We'll monitor:
   - Request rate (RPS)
   - Latency (P50, P95, P99)
   - Error rate
   - Resource utilization"

❌ "We'll monitor the system"
```

**Tip 2: Discuss Alerting Strategy**
```
✅ "We'll set up alerts:
   - Error rate > 1% → HIGH severity
   - Latency P99 > 500ms → MEDIUM severity
   - CPU > 90% → CRITICAL severity"

❌ "We'll alert on issues"
```

**Tip 3: Explain Monitoring Tools**
```
✅ "We'll use:
   - Prometheus for metrics
   - ELK stack for logs
   - Jaeger for distributed tracing
   - Grafana for visualization"

❌ "We'll use monitoring tools"
```

---

## 2. Iterate: Refine Design Based on Feedback

### 2.1 Why Iteration is Important

**Benefits**:
- Improves design quality
- Addresses concerns early
- Shows adaptability
- Demonstrates collaboration
- Leads to better solutions

**Common Mistakes**:
- Being too attached to initial design
- Not listening to feedback
- Defending design instead of improving
- Not asking for feedback

### 2.2 Iteration Process

#### Step 1: Present Initial Design

```
1. High-level architecture
2. Key components
3. Data flow
4. Trade-offs
5. Assumptions
```

#### Step 2: Gather Feedback

```
Listen for:
- Concerns
- Questions
- Suggestions
- Alternative approaches
- Missing considerations
```

#### Step 3: Refine Design

```
Address:
- Concerns raised
- Questions asked
- Suggestions made
- Alternatives considered
- Missing pieces
```

#### Step 4: Validate

```
Check:
- Does it address feedback?
- Are there new issues?
- Is it better?
- Any other concerns?
```

### 2.3 Common Feedback Scenarios

#### Scenario 1: Scalability Concern

```
INTERVIEWER: "What if you have 10x more users?"

YOUR RESPONSE:
"Good point. Let me refine the design:

Current: 10 servers
10x: 100 servers

But we can optimize:
1. Add more caching (reduce DB load)
2. Use read replicas (scale reads)
3. Implement sharding (scale writes)
4. Use CDN (reduce server load)

This should handle 10x with fewer servers."
```

#### Scenario 2: Consistency Concern

```
INTERVIEWER: "What about data consistency?"

YOUR RESPONSE:
"You're right. Let me reconsider:

Current: Eventual consistency
Issue: May show stale data

For critical data (payments, inventory):
- Use strong consistency
- Synchronous replication
- ACID transactions

For non-critical data (likes, views):
- Keep eventual consistency
- Better performance

Hybrid approach based on data criticality."
```

#### Scenario 3: Cost Concern

```
INTERVIEWER: "This seems expensive. Can we optimize cost?"

YOUR RESPONSE:
"Absolutely. Let me optimize:

Current: 100 servers × $100 = $10K/month

Optimizations:
1. Use reserved instances (40% discount)
2. Auto-scale based on load
3. Use spot instances for non-critical
4. Optimize caching (fewer DB calls)

Optimized: 50 servers × $60 = $3K/month
Savings: 70%"
```

#### Scenario 4: Missing Component

```
INTERVIEWER: "What about rate limiting?"

YOUR RESPONSE:
"Great catch! I missed that. Let me add:

Rate Limiting:
- At API Gateway level
- Per user/IP
- Token bucket algorithm
- Distributed rate limiting (Redis)

This prevents abuse and DDoS attacks."
```

### 2.4 How to Handle Feedback

#### A. Listen Actively

```
✅ "That's a good point. Let me think about that..."
✅ "I hadn't considered that. Let me refine..."
✅ "You're right. We should address that..."

❌ "Actually, my design already handles that..."
❌ "That's not necessary because..."
❌ "I disagree..."
```

#### B. Ask Clarifying Questions

```
✅ "Are you concerned about scalability or consistency?"
✅ "Should I focus on latency or throughput?"
✅ "What's the priority here?"

❌ (Assume what they mean)
```

#### C. Acknowledge and Improve

```
✅ "You're right. Let me update the design to address that..."
✅ "That's a valid concern. Here's how we can handle it..."
✅ "Good point. Let me refine this part..."

❌ (Ignore feedback)
```

### 2.5 Iteration Examples

#### Example 1: Adding Caching

```
INITIAL DESIGN:
Client → API → Database

FEEDBACK: "Database might be a bottleneck"

ITERATION 1:
Client → API → Cache → Database

FEEDBACK: "What about cache invalidation?"

ITERATION 2:
Client → API → Cache (with TTL)
                ↓ (miss)
            Database
                ↓
            Update cache

FEEDBACK: "What if cache fails?"

ITERATION 3:
Client → API → Cache (with fallback)
                ↓ (miss/fail)
            Database
                ↓
            Update cache (if available)

Final: Resilient caching with fallback
```

#### Example 2: Improving Scalability

```
INITIAL DESIGN:
Single database server

FEEDBACK: "Won't scale"

ITERATION 1:
Master-Slave replication
- Master: writes
- Slaves: reads

FEEDBACK: "What about write scaling?"

ITERATION 2:
Sharded database
- 10 shards
- Hash-based sharding
- Each shard: Master-Slave

FEEDBACK: "What about cross-shard queries?"

ITERATION 3:
Sharded database + Denormalization
- Shard by user_id
- Denormalize data to avoid joins
- Use materialized views for analytics

Final: Scalable, sharded database design
```

### 2.6 Interview Tips

**Tip 1: Be Open to Feedback**
```
✅ "That's a great point. Let me refine the design..."
✅ "I hadn't considered that. Let me think..."

❌ "Actually, my design already handles that..."
❌ "That's not necessary..."
```

**Tip 2: Ask for Feedback**
```
✅ "Does this architecture make sense?"
✅ "Should I consider anything else?"
✅ "What are your thoughts on this approach?"

❌ (Don't ask, just present)
```

**Tip 3: Show Iteration**
```
✅ "Based on your feedback, let me update:
   - Add caching layer
   - Implement sharding
   - Add rate limiting"

❌ (Stick to original design)
```

**Tip 4: Validate Changes**
```
✅ "With these changes:
   - Latency: 200ms → 50ms
   - Throughput: 1K → 10K req/sec
   - Cost: $10K → $5K/month"

❌ "This should be better"
```

### 2.7 Complete Iteration Example

**Problem**: Design a URL Shortener

**Initial Design**:
```
Client → API → Database
```

**Iteration 1 - Feedback: "Database bottleneck"**
```
Client → API → Cache → Database
```

**Iteration 2 - Feedback: "How to generate unique IDs?"**
```
Client → API → ID Generator (Snowflake)
                ↓
            Cache → Database
```

**Iteration 3 - Feedback: "What about analytics?"**
```
Client → API → ID Generator
                ↓
            Cache → Database
                ↓
            Analytics Service → Analytics DB
```

**Iteration 4 - Feedback: "How to handle high read traffic?"**
```
Client → CDN (cached redirects)
         ↓ (miss)
         API → Cache → Database
         ↓
         Analytics Service
```

**Final Design**: Scalable, cached, with analytics

---

## Summary

### Monitoring
1. ✅ Monitor key metrics (latency, traffic, errors, saturation)
2. ✅ Implement structured logging
3. ✅ Set up alerting (thresholds, severity)
4. ✅ Use distributed tracing
5. ✅ Visualize with dashboards

### Iterate
1. ✅ Present initial design
2. ✅ Listen to feedback actively
3. ✅ Refine based on feedback
4. ✅ Validate improvements
5. ✅ Show iteration process

**Key Takeaway**: Monitoring ensures you can operate the system effectively, and iteration ensures you build the right system. Both demonstrate senior-level thinking and production experience.

