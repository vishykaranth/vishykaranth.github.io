# System Design Interview Tips - Part 2: Think Scalable & Consider Trade-offs

## Overview

This document provides in-depth guidance on scalability thinking and trade-off analysis in system design interviews:
1. **Think Scalable**: Design for 10x, 100x scale
2. **Consider Trade-offs**: Consistency vs Availability, Latency vs Throughput

---

## 1. Think Scalable: Design for 10x, 100x Scale

### 1.1 Why Scalability Matters

**Real-World Examples**:
- Twitter: 500M tweets/day → 200B tweets/day (400x growth)
- Facebook: 1B users → 3B users (3x growth)
- Netflix: 100M subscribers → 200M+ subscribers (2x+ growth)

**Interview Context**:
- Shows forward-thinking
- Demonstrates understanding of growth
- Identifies bottlenecks early
- Shows experience with scale

### 1.2 Scalability Dimensions

#### A. Horizontal vs Vertical Scaling

**Vertical Scaling (Scale Up)**:
```
Single Server:
CPU: 4 cores → 8 cores → 16 cores
RAM: 16GB → 32GB → 64GB
Storage: 1TB → 2TB → 4TB

Limitations:
- Hardware limits
- Single point of failure
- Cost increases exponentially
```

**Horizontal Scaling (Scale Out)**:
```
Multiple Servers:
1 server → 10 servers → 100 servers → 1000 servers

Advantages:
- Linear cost scaling
- Fault tolerance
- No hardware limits
```

**When to Use**:
- **Vertical**: Small scale, simple architecture, stateful applications
- **Horizontal**: Large scale, stateless services, distributed systems

#### B. Read Scaling vs Write Scaling

**Read Scaling**:
```
Problem: 1000 reads/sec per server
Solution: Add read replicas

1 Master → 10 Read Replicas
Read capacity: 1000 → 10,000 reads/sec
```

**Write Scaling**:
```
Problem: 1000 writes/sec per server
Solution: Sharding/Partitioning

1 Database → 10 Shards
Write capacity: 1000 → 10,000 writes/sec
```

### 1.3 Scalability Patterns

#### Pattern 1: Stateless Services

```
❌ Stateful (Doesn't Scale):
Server 1: User session stored in memory
Server 2: Doesn't know about session
→ Can't load balance effectively

✅ Stateless (Scales):
Server 1, 2, 3: No local state
Session stored in Redis
→ Any server can handle any request
```

**Example: Web Application**

```
Stateless Design:
┌─────────┐
│ Client  │
└────┬────┘
     │
     ├───→ Server 1 (no state)
     ├───→ Server 2 (no state)
     └───→ Server 3 (no state)
           │
           └───→ Redis (shared state)
```

#### Pattern 2: Database Sharding

```
Problem: Single database can't handle 1M writes/sec

Solution: Shard by user_id

Shard 1: user_id % 10 == 0
Shard 2: user_id % 10 == 1
...
Shard 10: user_id % 10 == 9

Each shard handles 100K writes/sec
Total: 1M writes/sec
```

**Sharding Strategies**:

1. **Range-Based Sharding**:
```
Shard 1: user_id 1-1M
Shard 2: user_id 1M-2M
Shard 3: user_id 2M-3M
```

2. **Hash-Based Sharding**:
```
Shard = hash(user_id) % num_shards
```

3. **Directory-Based Sharding**:
```
Lookup table: user_id → shard_id
```

#### Pattern 3: Caching Strategy

```
Problem: Database can't handle 100K reads/sec

Solution: Multi-level caching

Level 1: Application cache (in-memory)
Level 2: Distributed cache (Redis)
Level 3: CDN (for static content)
Level 4: Database

Cache hit rate: 90%
→ Database load: 100K → 10K reads/sec
```

**Caching Layers**:

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
┌──────▼──────┐  Cache Hit: 60%
│   CDN       │  (Static content)
└──────┬──────┘
       │
┌──────▼──────┐  Cache Hit: 25%
│   Redis     │  (Hot data)
└──────┬──────┘
       │
┌──────▼──────┐  Cache Hit: 5%
│ App Cache   │  (Very hot data)
└──────┬──────┘
       │
┌──────▼──────┐
│  Database   │  (10% of requests)
└─────────────┘
```

#### Pattern 4: Load Balancing

```
Problem: Single server can't handle traffic

Solution: Load balancer + multiple servers

┌─────────┐
│ Client  │
└────┬────┘
     │
┌────▼────────┐
│Load Balancer│
└────┬────────┘
     │
     ├───→ Server 1
     ├───→ Server 2
     ├───→ Server 3
     └───→ Server N
```

**Load Balancing Algorithms**:
- Round Robin
- Least Connections
- Weighted Round Robin
- IP Hash (for session affinity)

#### Pattern 5: Database Replication

```
Problem: Single database is bottleneck

Solution: Master-Slave Replication

Master: Handles all writes
Slaves: Handle reads (10x read capacity)

┌─────────┐
│ Writes  │──→ Master DB
└─────────┘
     │
     ├───→ Replicate
     │
     ├───→ Slave 1 (Reads)
     ├───→ Slave 2 (Reads)
     └───→ Slave N (Reads)
```

### 1.4 Scaling by Component

#### A. Scaling Web Servers

```
Initial: 1 server, 1000 req/sec
10x: 10 servers, 10,000 req/sec
100x: 100 servers, 100,000 req/sec

Key: Stateless design, load balancer
```

#### B. Scaling Application Servers

```
Initial: 1 server, 500 req/sec
10x: 10 servers, 5,000 req/sec
100x: 100 servers, 50,000 req/sec

Key: Stateless, connection pooling, async processing
```

#### C. Scaling Databases

```
Read Scaling:
Initial: 1 DB, 10K reads/sec
10x: 1 Master + 10 Replicas, 100K reads/sec
100x: 1 Master + 100 Replicas, 1M reads/sec

Write Scaling:
Initial: 1 DB, 1K writes/sec
10x: 10 Shards, 10K writes/sec
100x: 100 Shards, 100K writes/sec
```

#### D. Scaling Caches

```
Initial: 1 Redis instance, 50K ops/sec
10x: Redis Cluster (10 nodes), 500K ops/sec
100x: Redis Cluster (100 nodes), 5M ops/sec

Key: Consistent hashing, sharding
```

### 1.5 Scalability Calculation Example

**Problem**: Design a system for 1M users, scale to 100M users

**Initial Scale (1M users)**:
```
DAU: 500K (50% of users)
Requests per user per day: 100
Total requests/day: 50M
Requests/sec: 50M / 86400 = ~580 req/sec
Peak (10x): 5,800 req/sec

Storage: 1M users * 1KB/user = 1GB
Growth: 10% per month
```

**10x Scale (10M users)**:
```
DAU: 5M
Requests/sec: 5,800 req/sec
Peak: 58,000 req/sec

Storage: 10GB
Servers: 10x (10 web servers, 10 app servers)
Database: Sharded (10 shards)
Cache: Redis Cluster (10 nodes)
```

**100x Scale (100M users)**:
```
DAU: 50M
Requests/sec: 58,000 req/sec
Peak: 580,000 req/sec

Storage: 100GB
Servers: 100x (100 web servers, 100 app servers)
Database: Sharded (100 shards)
Cache: Redis Cluster (100 nodes)
CDN: Required for static content
```

### 1.6 Identifying Bottlenecks

**Common Bottlenecks**:

1. **Database**:
   - Single database can't scale writes
   - Solution: Sharding, read replicas

2. **Network**:
   - Bandwidth limitations
   - Solution: CDN, compression

3. **CPU**:
   - CPU-intensive operations
   - Solution: Horizontal scaling, async processing

4. **Memory**:
   - Memory constraints
   - Solution: Caching, memory optimization

5. **I/O**:
   - Disk I/O bottlenecks
   - Solution: SSDs, caching, async I/O

**Bottleneck Analysis**:

```
For each component, ask:
1. What's the current capacity?
2. What's the bottleneck?
3. How to scale 10x?
4. How to scale 100x?
```

### 1.7 Scalability Best Practices

**Practice 1: Design for Scale from Start**
```
❌ "We'll add caching later"
✅ "We'll use caching from the start"
```

**Practice 2: Measure and Monitor**
```
- Monitor key metrics
- Identify bottlenecks early
- Plan capacity ahead
```

**Practice 3: Use Appropriate Patterns**
```
- Stateless services
- Database sharding
- Caching layers
- CDN for static content
```

**Practice 4: Plan for Growth**
```
- Design for 10x initially
- Plan for 100x growth
- Make scaling incremental
```

---

## 2. Consider Trade-offs: Consistency vs Availability, Latency vs Throughput

### 2.1 Why Trade-offs Matter

**Reality**: No perfect solution exists. Every design decision involves trade-offs.

**Common Trade-offs**:
- Consistency vs Availability
- Latency vs Throughput
- Cost vs Performance
- Complexity vs Simplicity
- Flexibility vs Optimization

### 2.2 CAP Theorem

**CAP Theorem**: In a distributed system, you can only guarantee 2 out of 3:

```
C - Consistency: All nodes see same data
A - Availability: System remains operational
P - Partition Tolerance: System continues despite network partitions
```

**Since network partitions are inevitable, you must choose**:

1. **CP (Consistency + Partition Tolerance)**:
   - Strong consistency
   - May sacrifice availability during partitions
   - Examples: Traditional databases, financial systems

2. **AP (Availability + Partition Tolerance)**:
   - High availability
   - May sacrifice consistency (eventual consistency)
   - Examples: DNS, CDN, NoSQL databases (Cassandra)

3. **CA (Consistency + Availability)**:
   - Not possible in distributed systems
   - Only works in single-node systems

**Visual Representation**:

```
        Consistency
            │
            │
    CP ─────┼───── CA (Not possible)
            │
            │
    ────────┼──────── Availability
            │
            │
            │
    Partition Tolerance
```

### 2.3 Consistency vs Availability

#### A. Strong Consistency (CP)

**Characteristics**:
- All reads return latest write
- Synchronous replication
- Higher latency
- Lower availability during partitions

**Use Cases**:
- Financial transactions
- Payment systems
- Inventory management
- User account balances

**Example: Bank Account Transfer**

```
Account A: $1000
Account B: $500

Transfer $100 from A to B

With Strong Consistency:
1. Lock Account A
2. Lock Account B
3. Debit A: $1000 - $100 = $900
4. Credit B: $500 + $100 = $600
5. Commit transaction
6. Unlock accounts

Both accounts always show correct balance
```

**Trade-offs**:
- ✅ Data always correct
- ✅ No stale data
- ❌ Higher latency (locks, synchronous writes)
- ❌ Lower availability (blocks during partitions)

#### B. Eventual Consistency (AP)

**Characteristics**:
- Reads may return stale data initially
- Asynchronous replication
- Lower latency
- Higher availability

**Use Cases**:
- Social media feeds
- Comments/likes
- User profiles
- News feeds

**Example: Social Media Like**

```
Post has 1000 likes

User A likes → Replica 1: 1001 likes
User B likes → Replica 2: 1001 likes

Replication happens asynchronously:
Replica 1 → Replica 2: 1001 likes
Replica 2 → Replica 1: 1001 likes

Eventually, both show 1001 likes
```

**Trade-offs**:
- ✅ High availability
- ✅ Low latency
- ✅ Better performance
- ❌ May show stale data temporarily
- ❌ Conflict resolution needed

#### C. Choosing Consistency Level

**Decision Framework**:

```
Ask: "What happens if data is stale?"

Critical (Strong Consistency):
- Money transfers
- Inventory counts
- Medical records
- Legal documents

Non-Critical (Eventual Consistency):
- Social media likes
- View counts
- Recommendations
- User preferences
```

### 2.4 Latency vs Throughput

#### A. Low Latency

**Characteristics**:
- Fast response times
- Optimized for individual requests
- May sacrifice throughput

**Use Cases**:
- Real-time systems
- Gaming
- Trading systems
- Video conferencing

**Example: Real-Time Chat**

```
Requirement: Message delivery < 100ms

Optimizations:
- Direct WebSocket connections
- In-memory message queue
- No database writes (async)
- Minimal processing

Trade-off:
- Low latency: < 100ms ✅
- Lower throughput: 10K msg/sec
```

#### B. High Throughput

**Characteristics**:
- High requests per second
- Optimized for batch processing
- May sacrifice latency

**Use Cases**:
- Analytics
- Batch processing
- Data pipelines
- Reporting

**Example: Analytics System**

```
Requirement: Process 1M events/sec

Optimizations:
- Batch processing
- Async writes
- Buffering
- Parallel processing

Trade-off:
- High throughput: 1M events/sec ✅
- Higher latency: 1-5 seconds
```

#### C. Balancing Latency and Throughput

**Strategies**:

1. **Async Processing**:
```
Request → Immediate response (low latency)
        → Async processing (high throughput)
```

2. **Caching**:
```
Cache hot data → Low latency
Batch cold data → High throughput
```

3. **Prioritization**:
```
Critical requests → Low latency path
Non-critical → High throughput path
```

### 2.5 Other Important Trade-offs

#### A. Cost vs Performance

```
High Performance:
- More servers
- Better hardware
- Premium services
→ Higher cost

Cost Optimization:
- Fewer servers
- Standard hardware
- Reserved instances
→ Lower performance
```

#### B. Complexity vs Simplicity

```
Simple Design:
- Easy to understand
- Easy to maintain
- Limited features
→ May not scale

Complex Design:
- More features
- Better scalability
- Harder to maintain
→ Higher complexity
```

#### C. Flexibility vs Optimization

```
Flexible Design:
- Supports many use cases
- Easy to extend
- General-purpose
→ May not be optimal

Optimized Design:
- Specific use case
- High performance
- Hard to change
→ Less flexible
```

### 2.6 Trade-off Analysis Framework

**Step 1: Identify Trade-offs**

```
For each design decision:
1. What are the alternatives?
2. What are the trade-offs?
3. What are the constraints?
```

**Step 2: Evaluate Options**

```
Option A:
- Pros: [list]
- Cons: [list]
- Best for: [scenario]

Option B:
- Pros: [list]
- Cons: [list]
- Best for: [scenario]
```

**Step 3: Make Decision**

```
Based on:
- Requirements
- Constraints
- Priorities
- Future needs
```

### 2.7 Example: Trade-off Analysis

**Problem**: Design a social media feed system

**Trade-off 1: Consistency vs Availability**

```
Option A: Strong Consistency
- All users see same feed
- Synchronous updates
- Lower availability during failures
- Higher latency

Option B: Eventual Consistency
- Users may see slightly different feeds
- Asynchronous updates
- Higher availability
- Lower latency

Decision: Eventual Consistency
Reason: Freshness not critical, availability is
```

**Trade-off 2: Latency vs Throughput**

```
Option A: Real-time Feed Generation
- Generate feed on-demand
- Low latency: < 200ms
- Lower throughput: 1K req/sec per server

Option B: Pre-computed Feeds
- Pre-compute and cache feeds
- Higher latency: 500ms (cache miss)
- Higher throughput: 10K req/sec per server

Decision: Hybrid Approach
- Pre-compute for 80% of users (cache hit: < 50ms)
- On-demand for 20% (cache miss: < 200ms)
```

**Trade-off 3: Cost vs Performance**

```
Option A: More Servers
- 100 servers
- High performance
- Cost: $10K/month

Option B: Fewer Servers + Caching
- 20 servers + Redis cluster
- Good performance
- Cost: $3K/month

Decision: Option B
Reason: 70% cost savings, acceptable performance
```

### 2.8 Interview Tips for Trade-offs

**Tip 1: Always Mention Trade-offs**
```
✅ "We're using eventual consistency here, which means
   we get high availability but may show slightly stale data"

❌ "We're using eventual consistency"
```

**Tip 2: Explain Your Choice**
```
✅ "Given that freshness is not critical and availability
   is important, we chose eventual consistency"

❌ "We chose eventual consistency"
```

**Tip 3: Discuss Alternatives**
```
✅ "We could use strong consistency, but that would
   increase latency and reduce availability. Given our
   requirements, eventual consistency is better"

❌ "We're using eventual consistency"
```

**Tip 4: Quantify Trade-offs**
```
✅ "Eventual consistency gives us 99.9% availability vs
   99% with strong consistency, but may show data that's
   up to 5 seconds old"

❌ "Eventual consistency is better"
```

---

## Summary

### Think Scalable
1. ✅ Design for 10x, 100x growth
2. ✅ Use horizontal scaling patterns
3. ✅ Identify bottlenecks early
4. ✅ Plan capacity ahead
5. ✅ Use appropriate scaling patterns

### Consider Trade-offs
1. ✅ Understand CAP theorem
2. ✅ Choose consistency level appropriately
3. ✅ Balance latency vs throughput
4. ✅ Evaluate cost vs performance
5. ✅ Explain trade-offs clearly

**Key Takeaway**: Scalability and trade-offs are fundamental to system design. Always think about growth and be explicit about the trade-offs you're making.

