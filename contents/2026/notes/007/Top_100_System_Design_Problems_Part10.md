# Top 100 System Design Problems with Database Selection Criteria - Part 10

## Problems 46-50: Specialized Systems

### Problem 46: Design a Distributed Counter

#### Requirements
- **Scale**: 1B+ counters, 1M+ increments/sec, distributed updates
- **Features**: Increment/decrement, consistency, sharding, aggregation
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Counter values, timestamps, metadata
- **Consistency**: Configurable (strong or eventual)

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Atomic Operations**: INCR, DECR for atomic increments
- **Low Latency**: Sub-millisecond counter operations
- **High Throughput**: Millions of operations per second
- **Data Structures**: Strings for counters, sorted sets for ranked counters
- **Use Case**: Real-time counters, view counts, like counts

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Persistent counters, aggregated counters, metadata
   - **Why**: ACID transactions, complex queries, aggregations
   - **Schema**: Counters, metadata, aggregated values

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Distributed counters, high-scale counters
   - **Why**: High write throughput, horizontal scaling, counters support
   - **Schema**: `counters_by_key`, distributed counters

3. **DynamoDB** (NoSQL - Key-Value)
   - **Use Case**: Managed distributed counters
   - **Why**: Managed service, atomic increments, auto-scaling
   - **Schema**: Counters by key with atomic operations

4. **TimescaleDB** (Time-Series)
   - **Use Case**: Time-based counters, counter history, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Counter values over time, aggregations

5. **MongoDB** (Document Store)
   - **Use Case**: Counter metadata, nested counter data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Counter documents with metadata

**Database Architecture**:
```
Counter Update → Counter Service
    ├── Redis → Real-time counters, atomic operations
    ├── PostgreSQL → Persistent counters, aggregated values
    ├── Cassandra → Distributed counters, high-scale
    ├── DynamoDB → Managed counters
    ├── TimescaleDB → Counter history, analytics
    └── MongoDB → Counter metadata
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | DynamoDB | TimescaleDB |
|------------|-------|------------|-----------|----------|-------------|
| Atomic Increments | ✅ Excellent | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Good | ⚠️ Moderate |
| High Throughput | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Distributed Counters | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ✅ Excellent | ❌ Poor |
| Counter History | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |

---

### Problem 47: Design a Distributed Pub-Sub System (Kafka)

#### Requirements
- **Scale**: 1B+ messages/day, 1M+ messages/sec, thousands of topics
- **Features**: Message publishing, subscription, partitioning, replication
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Messages, topics, partitions, consumer groups, offsets
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Kafka (Distributed Message Queue)**

**Why**:
- **Distributed**: Partitioned topics across brokers
- **High Throughput**: Millions of messages per second
- **Replication**: Multi-replica for fault tolerance
- **Consumer Groups**: Parallel consumption, offset management
- **Use Case**: Event streaming, pub-sub, message queuing

**Secondary Databases**:

1. **ZooKeeper** (Coordination)
   - **Use Case**: Broker coordination, topic metadata, consumer groups
   - **Why**: Distributed consensus, coordination
   - **Schema**: Broker nodes, topic metadata, consumer groups

2. **PostgreSQL** (Relational)
   - **Use Case**: Topic metadata, consumer metadata, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Topics, consumers, configurations, metadata

3. **Redis** (In-Memory Cache)
   - **Use Case**: Recent messages cache, consumer offsets cache
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Recent messages (5 min), offsets cache (1 min)

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Message search, log analytics, event search
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Message documents, event documents

5. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Long-term message storage, archival
   - **Why**: Cost-effective, durable storage
   - **Retention**: Messages older than retention period

**Database Architecture**:
```
Message → Producer → Kafka Cluster
    ├── Kafka → Message streaming, partitioning, replication
    ├── ZooKeeper → Broker coordination, metadata
    ├── PostgreSQL → Topic metadata, consumer metadata
    ├── Redis → Recent messages cache, offsets cache
    ├── Elasticsearch → Message search, analytics
    └── S3 → Long-term storage, archival
```

**Selection Matrix**:
| Requirement | Kafka | ZooKeeper | PostgreSQL | Redis | Elasticsearch |
|------------|-------|-----------|------------|-------|---------------|
| Message Streaming | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Partitioning | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Replication | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Consumer Groups | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Message Search | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 48: Design a Distributed Queue (SQS)

#### Requirements
- **Scale**: 1B+ messages/day, 100K+ messages/sec, millions of queues
- **Features**: Message queuing, delivery guarantees, visibility timeout, dead-letter
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Messages, queues, metadata, delivery receipts
- **Consistency**: At-least-once delivery

#### Database Selection Criteria

**Primary Database: RabbitMQ/Kafka (Message Queue)**

**Why**:
- **Message Queuing**: Reliable message storage and delivery
- **Delivery Guarantees**: At-least-once, exactly-once options
- **Visibility Timeout**: Message visibility management
- **Dead-Letter Queues**: Failed message handling
- **Use Case**: Task queues, message queuing, async processing

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Queue metadata cache, recent messages, priority queues
   - **Why**: Sub-millisecond latency, sorted sets for priority
   - **TTL**: Queue metadata (1 min), recent messages (5 min)

2. **PostgreSQL** (Relational)
   - **Use Case**: Queue metadata, message metadata, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Queues, messages, configurations, metadata

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Message storage (alternative), message history
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `messages_by_queue`, `messages_by_timestamp`

4. **DynamoDB** (NoSQL - Key-Value)
   - **Use Case**: Managed queue alternative, simple queuing
   - **Why**: Managed service, auto-scaling, low latency
   - **Schema**: Messages by queue, delivery receipts

5. **MongoDB** (Document Store)
   - **Use Case**: Message documents, nested message data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Message documents with payload, metadata

**Database Architecture**:
```
Message → Producer → Message Queue
    ├── RabbitMQ/Kafka → Message queuing, delivery
    ├── Redis → Queue metadata cache, priority queues
    ├── PostgreSQL → Queue metadata, configurations
    ├── Cassandra → Message storage (alternative)
    ├── DynamoDB → Managed queue alternative
    └── MongoDB → Message documents
```

**Selection Matrix**:
| Requirement | RabbitMQ/Kafka | Redis | PostgreSQL | Cassandra | DynamoDB |
|------------|----------------|-------|------------|-----------|----------|
| Message Queuing | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Good |
| Delivery Guarantees | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| Visibility Timeout | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| Dead-Letter Queue | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| Priority Queues | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate |

---

### Problem 49: Design a Distributed Tracing System (Jaeger/Zipkin)

#### Requirements
- **Scale**: 1B+ traces/day, 10M+ spans/sec, petabyte-scale storage
- **Features**: Trace collection, storage, querying, visualization
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Traces, spans, metadata, timestamps, relationships
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Elasticsearch (Search Engine)**

**Why**:
- **Trace Storage**: Document-based trace storage
- **Full-Text Search**: Search across traces, spans
- **Aggregations**: Trace analytics, aggregations
- **Time-Based Queries**: Efficient time-range queries
- **Use Case**: Trace storage, search, analytics

**Secondary Databases**:

1. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Trace storage (alternative), high-scale storage
   - **Why**: High write throughput, horizontal scaling, time-series data
   - **Schema**: `traces_by_service`, `spans_by_trace`, `traces_by_timestamp`

2. **TimescaleDB** (Time-Series)
   - **Use Case**: Trace metrics, performance metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Trace metrics over time, performance data

3. **Redis** (In-Memory Cache)
   - **Use Case**: Recent traces cache, trace metadata cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Recent traces (1 hour), metadata (5 min)

4. **PostgreSQL** (Relational)
   - **Use Case**: Trace metadata, service metadata, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Services, traces metadata, configurations

5. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Long-term trace storage, archival
   - **Why**: Cost-effective, durable storage
   - **Retention**: Traces older than retention period

**Database Architecture**:
```
Span → Tracing Agent → Collector → Storage
    ├── Elasticsearch → Trace storage, search, analytics
    ├── Cassandra → Trace storage (alternative), high-scale
    ├── TimescaleDB → Trace metrics, performance data
    ├── Redis → Recent traces cache, metadata
    ├── PostgreSQL → Trace metadata, service metadata
    └── S3 → Long-term storage, archival
```

**Selection Matrix**:
| Requirement | Elasticsearch | Cassandra | TimescaleDB | Redis | PostgreSQL |
|------------|---------------|-----------|-------------|-------|------------|
| Trace Storage | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Trace Search | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| High Write Throughput | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Trace Analytics | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Long-Term Storage | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |

---

### Problem 50: Design a Distributed ID Generator (Snowflake)

#### Requirements
- **Scale**: 1B+ IDs/sec, globally unique, time-ordered, distributed
- **Features**: Unique ID generation, time-based ordering, scalability
- **Read/Write Ratio**: 0:1 (write-only)
- **Data Types**: IDs, timestamps, machine IDs, sequence numbers
- **Consistency**: Strong consistency required (uniqueness)

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Atomic Operations**: INCR for sequence numbers
- **Low Latency**: Sub-millisecond ID generation
- **High Throughput**: Millions of IDs per second
- **Data Structures**: Strings for counters, hashes for machine state
- **Use Case**: Sequence numbers, machine IDs, timestamp components

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Machine registration, ID metadata, persistent storage
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Machines, ID ranges, metadata

2. **ZooKeeper/etcd** (Coordination)
   - **Use Case**: Machine coordination, leader election, configuration
   - **Why**: Distributed consensus, coordination
   - **Schema**: Machine nodes, configuration, leader election

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: ID generation logs, audit trail (optional)
   - **Why**: High write throughput, time-series data
   - **Schema**: ID generation logs (if needed)

4. **MongoDB** (Document Store)
   - **Use Case**: Machine metadata, ID metadata
   - **Why**: Flexible schema, nested structures
   - **Schema**: Machine documents, ID metadata

**Database Architecture**:
```
ID Request → ID Generator
    ├── Redis → Sequence numbers, machine state
    ├── PostgreSQL → Machine registration, metadata
    ├── ZooKeeper → Machine coordination, leader election
    ├── Cassandra → ID logs (optional)
    └── MongoDB → Machine metadata
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | ZooKeeper | Cassandra | MongoDB |
|------------|-------|------------|-----------|-----------|---------|
| ID Generation | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| High Throughput | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Machine Coordination | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ❌ Poor |
| Uniqueness Guarantee | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |

---

## Database Selection Framework (Continued)

### Specialized System Patterns

**Pattern 1: Distributed Counters**
- **Requirement**: Atomic increments, high throughput
- **Databases**: Redis, Cassandra, DynamoDB
- **Use Cases**: View counts, like counts, real-time metrics

**Pattern 2: Message Queuing**
- **Requirement**: Reliable delivery, ordering, durability
- **Databases**: Kafka, RabbitMQ, SQS, Redis
- **Use Cases**: Task queues, event streaming, async processing

**Pattern 3: Distributed Tracing**
- **Requirement**: High write throughput, search, analytics
- **Databases**: Elasticsearch, Cassandra, Jaeger backend
- **Use Cases**: Microservices tracing, performance monitoring

**Pattern 4: ID Generation**
- **Requirement**: Uniqueness, time-ordered, high throughput
- **Databases**: Redis, PostgreSQL, custom algorithms
- **Use Cases**: Distributed ID generation, unique identifiers

---

## Summary

**Problems Covered (46-50)**:
46. Distributed Counter → Redis, PostgreSQL, Cassandra, DynamoDB, TimescaleDB
47. Distributed Pub-Sub → Kafka, ZooKeeper, PostgreSQL, Redis, Elasticsearch
48. Distributed Queue → RabbitMQ/Kafka, Redis, PostgreSQL, Cassandra, DynamoDB
49. Distributed Tracing → Elasticsearch, Cassandra, TimescaleDB, Redis, PostgreSQL
50. Distributed ID Generator → Redis, PostgreSQL, ZooKeeper, Cassandra, MongoDB

**Key Patterns Identified**:
- **Counters**: Redis for atomic operations and low latency
- **Message Queuing**: Kafka/RabbitMQ for reliable delivery
- **Tracing**: Elasticsearch for search and analytics
- **ID Generation**: Redis for high-throughput unique IDs
- **Coordination**: ZooKeeper/etcd for distributed consensus

**Next**: Problems 51-55 (Additional Social Systems)

