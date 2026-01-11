# System Design Interview Tips - Part 3: Discuss Alternatives & Estimate Capacity

## Overview

This document provides in-depth guidance on discussing alternatives and performing capacity estimation in system design interviews:
1. **Discuss Alternatives**: Mention different approaches and their trade-offs
2. **Estimate Capacity**: Back-of-envelope calculations

---

## 1. Discuss Alternatives: Mention Different Approaches and Their Trade-offs

### 1.1 Why Discussing Alternatives is Important

**Benefits**:
- Shows deep understanding
- Demonstrates analytical thinking
- Shows you've considered multiple solutions
- Helps interviewer understand your reasoning
- Often leads to better final design

**Common Mistakes**:
- Presenting only one solution
- Not explaining why you chose a particular approach
- Not mentioning alternatives at all
- Being too attached to initial idea

### 1.2 When to Discuss Alternatives

**Always Discuss Alternatives For**:
- Database choices (SQL vs NoSQL)
- Caching strategies
- Message queue choices
- Load balancing algorithms
- Consistency models
- Storage solutions
- Communication protocols

### 1.3 Framework for Discussing Alternatives

#### Step 1: Identify Alternatives

```
For each major component, ask:
"What are the different approaches we could use?"
```

#### Step 2: Evaluate Each Alternative

```
For each alternative:
- Pros
- Cons
- Use cases
- Trade-offs
- Cost implications
```

#### Step 3: Make Recommendation

```
Based on:
- Requirements
- Constraints
- Trade-offs
- Team expertise
```

### 1.4 Common Alternatives by Component

#### A. Database Alternatives

**SQL vs NoSQL**

```
SQL Databases (PostgreSQL, MySQL):
Pros:
- ACID transactions
- Strong consistency
- Complex queries (JOINs)
- Mature ecosystem

Cons:
- Harder to scale horizontally
- Schema changes are expensive
- May be overkill for simple use cases

Use Cases:
- Financial transactions
- User accounts
- Complex relationships

NoSQL Databases (MongoDB, Cassandra):
Pros:
- Horizontal scaling
- Flexible schema
- High write throughput
- Good for specific use cases

Cons:
- Eventual consistency
- Limited query capabilities
- May need multiple databases

Use Cases:
- User sessions
- Time-series data
- High-volume writes
```

**Example: Chat Application**

```
Option 1: PostgreSQL
- Pros: ACID, complex queries, relationships
- Cons: Harder to scale, write bottleneck
- Best for: Message metadata, user data

Option 2: MongoDB
- Pros: Flexible schema, horizontal scaling
- Cons: Eventual consistency, no JOINs
- Best for: Message content, user profiles

Option 3: Cassandra
- Pros: High write throughput, linear scaling
- Cons: Eventual consistency, limited queries
- Best for: Message storage, high volume

Recommendation: Hybrid
- PostgreSQL: User data, relationships
- MongoDB: Message content
- Redis: Active sessions, recent messages
```

#### B. Caching Alternatives

**Cache-Aside vs Write-Through vs Write-Back**

```
Cache-Aside (Lazy Loading):
Application checks cache first
- Cache hit: Return data
- Cache miss: Query DB, update cache

Pros:
- Simple to implement
- Cache only what's needed
- Handles cache failures gracefully

Cons:
- Cache miss penalty
- Stale data possible
- Two round trips on miss

Write-Through:
Write to cache and DB simultaneously

Pros:
- Cache always up-to-date
- Data durability guaranteed

Cons:
- Higher write latency
- Writes unnecessary data to cache

Write-Back (Write-Behind):
Write to cache, async write to DB

Pros:
- Low write latency
- Batch DB writes

Cons:
- Risk of data loss
- Complex to implement
- Cache failure = data loss
```

**Example: Product Catalog**

```
Option 1: Cache-Aside
- Best for: Read-heavy workloads
- Use: Product details, prices

Option 2: Write-Through
- Best for: Critical data
- Use: Inventory counts

Option 3: Write-Back
- Best for: High write volume
- Use: View counts, analytics

Recommendation: Cache-Aside for most,
Write-Through for critical data
```

#### C. Message Queue Alternatives

**Kafka vs RabbitMQ vs SQS**

```
Apache Kafka:
Pros:
- High throughput
- Distributed, fault-tolerant
- Message replay
- Multiple consumers

Cons:
- Complex setup
- Higher latency
- Overkill for simple use cases

Use Cases:
- Event streaming
- Log aggregation
- Real-time analytics

RabbitMQ:
Pros:
- Easy to use
- Multiple exchange types
- Good for complex routing

Cons:
- Lower throughput
- Single broker bottleneck
- No message replay

Use Cases:
- Task queues
- Request routing
- Simple pub-sub

AWS SQS:
Pros:
- Managed service
- Auto-scaling
- Pay per use

Cons:
- Vendor lock-in
- Limited features
- Higher latency

Use Cases:
- Cloud-native applications
- Simple queuing needs
```

**Example: Notification System**

```
Option 1: Kafka
- Best for: High volume, multiple consumers
- Use: Event streaming, analytics

Option 2: RabbitMQ
- Best for: Complex routing, priority queues
- Use: Email notifications, SMS

Option 3: SQS
- Best for: AWS-native, simple needs
- Use: Basic notifications

Recommendation: Kafka for high volume,
RabbitMQ for complex routing
```

#### D. Load Balancing Alternatives

**Round Robin vs Least Connections vs IP Hash**

```
Round Robin:
Distribute requests evenly

Pros:
- Simple
- Predictable
- Good for similar servers

Cons:
- Doesn't consider server load
- May overload slow servers

Least Connections:
Route to server with fewest connections

Pros:
- Better load distribution
- Handles varying server capacity

Cons:
- More complex
- Requires connection tracking

IP Hash:
Route based on client IP

Pros:
- Session affinity
- Predictable routing

Cons:
- Uneven distribution possible
- Doesn't handle server failures well
```

**Example: Web Application**

```
Option 1: Round Robin
- Best for: Stateless, similar servers
- Use: API servers

Option 2: Least Connections
- Best for: Varying server capacity
- Use: Application servers

Option 3: IP Hash
- Best for: Session affinity needed
- Use: Stateful services

Recommendation: Least Connections for
general use, IP Hash for session affinity
```

#### E. Storage Alternatives

**Object Storage vs Block Storage vs File Storage**

```
Object Storage (S3):
Pros:
- Scalable
- Cost-effective
- Good for unstructured data

Cons:
- Higher latency
- Not good for frequent updates

Use Cases:
- Images, videos
- Backups
- Static assets

Block Storage (EBS):
Pros:
- Low latency
- Good for databases
- Direct access

Cons:
- More expensive
- Limited scalability

Use Cases:
- Databases
- High-performance applications

File Storage (EFS):
Pros:
- Shared access
- POSIX-compliant
- Good for multiple servers

Cons:
- Higher latency
- More expensive

Use Cases:
- Shared file systems
- Content management
```

### 1.5 How to Present Alternatives

#### Format 1: Comparison Table

```
| Feature | Option A | Option B | Option C |
|---------|----------|----------|----------|
| Latency | 10ms     | 50ms     | 100ms    |
| Cost    | $100/mo  | $50/mo   | $200/mo  |
| Scalability | High | Medium   | Low      |
| Complexity | High | Medium   | Low      |
```

#### Format 2: Pros/Cons List

```
Option A: PostgreSQL
Pros:
- ACID transactions
- Strong consistency
- Complex queries

Cons:
- Harder to scale
- More expensive

Option B: MongoDB
Pros:
- Easy to scale
- Flexible schema

Cons:
- Eventual consistency
- Limited queries
```

#### Format 3: Decision Tree

```
Need ACID?
├─ Yes → SQL (PostgreSQL)
└─ No → Need high writes?
    ├─ Yes → NoSQL (Cassandra)
    └─ No → Need complex queries?
        ├─ Yes → SQL (PostgreSQL)
        └─ No → NoSQL (MongoDB)
```

### 1.6 Example: Complete Alternative Discussion

**Problem**: Design a URL Shortener

**Database Alternatives**:

```
Option 1: SQL Database (PostgreSQL)
Pros:
- ACID guarantees
- Strong consistency
- Complex queries (analytics)

Cons:
- Write bottleneck
- Harder to scale
- Overkill for simple key-value

Option 2: NoSQL (MongoDB)
Pros:
- Easy to scale
- Flexible schema
- Good for key-value

Cons:
- Eventual consistency
- May not need flexibility

Option 3: Key-Value Store (Redis)
Pros:
- Very fast
- Perfect for key-value
- In-memory

Cons:
- Limited persistence
- Memory constraints
- Cost

Recommendation: Hybrid
- Redis: Hot URLs (cache)
- PostgreSQL: All URLs (persistence)
- Reason: Fast reads, durable storage
```

**URL Generation Alternatives**:

```
Option 1: Auto-increment + Base62
Pros:
- Simple
- Sequential IDs
- No collisions

Cons:
- Predictable
- Single point of failure
- Hard to scale

Option 2: UUID + Base62
Pros:
- Unique
- No coordination needed
- Scalable

Cons:
- Longer URLs
- Random, not sequential

Option 3: Distributed ID Generator (Snowflake)
Pros:
- Unique
- Time-ordered
- Scalable

Cons:
- More complex
- Clock dependency

Recommendation: Option 3 (Snowflake)
Reason: Scalable, time-ordered, unique
```

### 1.7 Interview Tips

**Tip 1: Always Mention Alternatives**
```
✅ "We could use PostgreSQL or MongoDB. Let me explain
   the trade-offs..."

❌ "We'll use PostgreSQL"
```

**Tip 2: Explain Your Choice**
```
✅ "Given our requirements for ACID transactions and
   complex queries, PostgreSQL is the better choice"

❌ "PostgreSQL is better"
```

**Tip 3: Be Open to Feedback**
```
✅ "We could also use MongoDB if we prioritize
   scalability over consistency. What do you think?"

❌ "PostgreSQL is the only option"
```

**Tip 4: Quantify Differences**
```
✅ "PostgreSQL gives us ACID but limits us to 10K
   writes/sec, while MongoDB can handle 100K writes/sec
   but with eventual consistency"

❌ "PostgreSQL is better for consistency"
```

---

## 2. Estimate Capacity: Back-of-Envelope Calculations

### 2.1 Why Capacity Estimation Matters

**Benefits**:
- Shows practical thinking
- Helps identify bottlenecks
- Guides design decisions
- Demonstrates quantitative skills
- Required for real-world systems

**Common Mistakes**:
- Not doing calculations
- Wrong assumptions
- Not showing work
- Ignoring important factors

### 2.2 Capacity Estimation Framework

#### Step 1: Identify What to Estimate

```
Common metrics:
- Storage (bytes, GB, TB)
- Bandwidth (bps, Mbps, Gbps)
- Requests per second (RPS)
- Queries per second (QPS)
- Number of servers
- Cost
```

#### Step 2: Make Assumptions

```
Document assumptions:
- Number of users
- Usage patterns
- Data sizes
- Growth rates
```

#### Step 3: Calculate

```
Show calculations:
- Formula
- Substitutions
- Result
- Units
```

#### Step 4: Validate

```
Check reasonableness:
- Does it make sense?
- Compare to known systems
- Consider growth
```

### 2.3 Common Calculations

#### A. Storage Estimation

**Formula**:
```
Total Storage = Number of Records × Size per Record × Retention Period
```

**Example: URL Shortener**

```
Assumptions:
- 100M URLs/day
- 500 bytes per URL (short URL + long URL + metadata)
- 5 years retention

Calculation:
Daily: 100M × 500 bytes = 50GB/day
Monthly: 50GB × 30 = 1.5TB/month
Yearly: 1.5TB × 12 = 18TB/year
5 Years: 18TB × 5 = 90TB

With 3x replication: 90TB × 3 = 270TB
```

**Example: Social Media Posts**

```
Assumptions:
- 500M users
- 5 posts/user/day average
- 1KB per post (text + metadata)
- 30 days retention

Calculation:
Posts/day: 500M × 5 = 2.5B posts/day
Storage/day: 2.5B × 1KB = 2.5TB/day
30 days: 2.5TB × 30 = 75TB

With 3x replication: 75TB × 3 = 225TB
```

#### B. Bandwidth Estimation

**Formula**:
```
Bandwidth = Requests/sec × Average Response Size
```

**Example: Video Streaming**

```
Assumptions:
- 1M concurrent users
- Average bitrate: 5 Mbps
- Peak: 10x average

Calculation:
Average: 1M × 5 Mbps = 5 Tbps
Peak: 5 Tbps × 10 = 50 Tbps

Per server (10 Gbps): 50 Tbps / 10 Gbps = 5,000 servers
```

**Example: Image Service**

```
Assumptions:
- 100K requests/sec
- Average image: 200KB
- Peak: 5x average

Calculation:
Average: 100K × 200KB = 20 GB/sec = 160 Gbps
Peak: 160 Gbps × 5 = 800 Gbps

With CDN (90% cache hit):
Origin: 800 Gbps × 10% = 80 Gbps
CDN: 800 Gbps × 90% = 720 Gbps
```

#### C. Request Rate Estimation

**Formula**:
```
Requests/sec = (Users × Requests per user per day) / (24 × 3600)
Peak = Average × Peak multiplier
```

**Example: E-Commerce**

```
Assumptions:
- 10M users
- 10 requests/user/day average
- Peak: 10x average (Black Friday)

Calculation:
Average: (10M × 10) / 86400 = 1,157 req/sec
Peak: 1,157 × 10 = 11,570 req/sec

Per server capacity: 1,000 req/sec
Servers needed: 11,570 / 1,000 = ~12 servers
```

**Example: Search Engine**

```
Assumptions:
- 1B users
- 3 searches/user/day average
- Peak: 5x average

Calculation:
Average: (1B × 3) / 86400 = 34,722 req/sec
Peak: 34,722 × 5 = 173,610 req/sec

Per server capacity: 5,000 req/sec
Servers needed: 173,610 / 5,000 = ~35 servers
```

#### D. Database Capacity

**Formula**:
```
Read QPS = Total Reads / Time
Write QPS = Total Writes / Time
```

**Example: Social Media**

```
Assumptions:
- 500M users
- 100 reads/user/day (feed, profile, etc.)
- 10 writes/user/day (posts, likes, etc.)

Calculation:
Reads/day: 500M × 100 = 50B reads/day
Reads/sec: 50B / 86400 = 578,704 reads/sec

Writes/day: 500M × 10 = 5B writes/day
Writes/sec: 5B / 86400 = 57,870 writes/sec

With read replicas (10:1 ratio):
Master: 57,870 writes/sec
Replicas: 578,704 reads/sec / 10 = 57,870 reads/sec per replica
Number of replicas: 10
```

#### E. Cache Capacity

**Formula**:
```
Cache Size = Hot Data Size × Replication Factor
Cache Hit Rate = Cache Hits / Total Requests
```

**Example: Product Catalog**

```
Assumptions:
- 10M products
- 1% are hot (100K products)
- 10KB per product
- 80% cache hit rate

Calculation:
Hot data: 100K × 10KB = 1GB
With replication (3x): 1GB × 3 = 3GB

Cache requests: 1M req/sec
Cache hits: 1M × 80% = 800K req/sec
Cache misses: 1M × 20% = 200K req/sec
DB load: 200K req/sec (reduced from 1M)
```

### 2.4 Server Capacity Estimation

**Formula**:
```
Servers Needed = Total Capacity Required / Capacity per Server
```

**Example: Web Application**

```
Assumptions:
- 100K requests/sec
- Each server: 1,000 req/sec
- 3x redundancy (for high availability)

Calculation:
Base servers: 100K / 1,000 = 100 servers
With redundancy: 100 × 3 = 300 servers

Cost estimation:
- Server cost: $100/month
- Total: 300 × $100 = $30,000/month
```

### 2.5 Cost Estimation

**Example: Complete System**

```
Components:
1. Web Servers: 100 × $50 = $5,000/month
2. Application Servers: 50 × $100 = $5,000/month
3. Database: 10 × $200 = $2,000/month
4. Cache: 5 × $150 = $750/month
5. CDN: 10TB × $0.10/GB = $1,000/month
6. Storage: 100TB × $0.023/GB = $2,300/month

Total: ~$16,050/month
Annual: ~$192,600/year
```

### 2.6 Complete Example: URL Shortener

**Requirements**:
- 100M URLs/day
- 100:1 read-to-write ratio
- 5 years retention
- 99.9% availability

**Calculations**:

```
1. WRITE RATE
Writes/day: 100M
Writes/sec: 100M / 86400 = 1,157 writes/sec
Peak (10x): 11,570 writes/sec

2. READ RATE
Reads/day: 100M × 100 = 10B reads/day
Reads/sec: 10B / 86400 = 115,740 reads/sec
Peak (10x): 1,157,400 reads/sec

3. STORAGE
URL size: 500 bytes
Daily: 100M × 500 bytes = 50GB/day
5 years: 50GB × 365 × 5 = 91.25TB
With 3x replication: 273.75TB

4. DATABASE
Write capacity: 11,570 writes/sec
Read capacity: 1,157,400 reads/sec

Option 1: Single DB
- Can't handle writes (bottleneck)

Option 2: Sharded DB
- 10 shards: 1,157 writes/sec per shard ✅
- Read replicas: 10 per shard
- Total: 10 shards × 10 replicas = 100 DB instances

5. CACHE
Hot URLs: 20% of reads
Cache size: 20% × 1,157,400 = 231,480 req/sec
Cache hit rate: 80%
Cache hits: 231,480 × 80% = 185,184 req/sec
Cache misses: 231,480 × 20% = 46,296 req/sec
DB reads: 1,157,400 - 185,184 = 972,216 req/sec
With cache: 972,216 + 46,296 = 1,018,512 req/sec

6. SERVERS
API servers: 1,157,400 / 5,000 = 232 servers
With redundancy: 232 × 3 = 696 servers

7. BANDWIDTH
Average response: 500 bytes
Bandwidth: 1,157,400 × 500 bytes = 578.7 MB/sec = 4.6 Gbps
Peak: 4.6 Gbps × 10 = 46 Gbps

8. COST ESTIMATION
API servers: 696 × $50 = $34,800/month
Database: 100 × $200 = $20,000/month
Cache: 20 × $150 = $3,000/month
Storage: 274TB × $0.023/GB = $6,302/month
CDN: 50TB × $0.10/GB = $5,000/month
Total: ~$69,102/month
```

### 2.7 Interview Tips

**Tip 1: Show Your Work**
```
✅ "Let me calculate:
   100M URLs/day = 100M / 86400 = 1,157 URLs/sec"

❌ "About 1,000 URLs/sec"
```

**Tip 2: State Assumptions**
```
✅ "Assuming 500 bytes per URL and 5 years retention..."

❌ (No assumptions stated)
```

**Tip 3: Round Appropriately**
```
✅ "1,157 URLs/sec, let's round to 1,200 for safety"

❌ "1,157.407 URLs/sec"
```

**Tip 4: Validate Results**
```
✅ "1,200 URLs/sec seems reasonable compared to
   Twitter's 6,000 tweets/sec"

❌ (No validation)
```

**Tip 5: Consider Growth**
```
✅ "At 1,200 URLs/sec, we need to plan for 10x growth
   to 12,000 URLs/sec"

❌ "1,200 URLs/sec is fine"
```

---

## Summary

### Discuss Alternatives
1. ✅ Identify multiple approaches
2. ✅ Evaluate pros/cons
3. ✅ Explain your choice
4. ✅ Be open to feedback
5. ✅ Quantify differences

### Estimate Capacity
1. ✅ Identify metrics to estimate
2. ✅ State assumptions clearly
3. ✅ Show calculations
4. ✅ Validate results
5. ✅ Consider growth

**Key Takeaway**: Discussing alternatives shows depth of thinking, and capacity estimation ensures your design is practical and scalable.

