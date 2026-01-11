# Top 100 System Design Problems with Database Selection Criteria - Part 9

## Problems 41-45: Infrastructure & DevOps Systems

### Problem 41: Design a Load Balancer

#### Requirements
- **Scale**: 1M+ requests/sec, thousands of backend servers, < 1ms latency
- **Features**: Request routing, health checks, session affinity, SSL termination
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Routing rules, health status, session data, metrics
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Session Affinity**: Store session-to-server mappings
- **Health Status**: Real-time health check results
- **Low Latency**: Sub-millisecond routing decisions
- **Data Structures**: Hashes for routing, sets for healthy servers
- **Use Case**: Session persistence, health status, routing cache

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Load balancer configuration, routing rules, backend servers
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Backend servers, routing rules, configurations

2. **etcd/Consul** (Configuration Store)
   - **Use Case**: Service discovery, configuration, health checks
   - **Why**: Distributed consensus, service discovery
   - **Schema**: Service registrations, health checks, configurations

3. **Redis** (In-Memory Cache)
   - **Use Case**: Session storage, health status cache
   - **Why**: Sub-millisecond access, high throughput
   - **TTL**: Session data (session duration), health status (10 sec)

4. **TimescaleDB** (Time-Series)
   - **Use Case**: Load balancer metrics, request metrics
   - **Why**: Time-series optimization, analytics
   - **Schema**: Request metrics, health check history

5. **Elasticsearch** (Search Engine)
   - **Use Case**: Request logs, access logs, analytics
   - **Why**: Full-text search, aggregations, log analysis
   - **Schema**: Access log documents, request documents

**Database Architecture**:
```
Request → Load Balancer
    ├── Redis → Session affinity, health status, routing cache
    ├── PostgreSQL → Configuration, routing rules, backend servers
    ├── etcd/Consul → Service discovery, configuration
    ├── TimescaleDB → Metrics, analytics
    └── Elasticsearch → Access logs, analytics
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | etcd/Consul | TimescaleDB | Elasticsearch |
|------------|-------|------------|-------------|-------------|---------------|
| Session Affinity | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Health Checks | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor |
| Routing Rules | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor |
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Metrics & Analytics | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ✅ Excellent |

---

### Problem 42: Design a Service Discovery System (Consul/etcd)

#### Requirements
- **Scale**: 10K+ services, 100K+ service instances, real-time updates
- **Features**: Service registration, health checks, service discovery, configuration
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Service registrations, health status, configurations, metadata
- **Consistency**: Strong consistency required (service discovery)

#### Database Selection Criteria

**Primary Database: etcd/Consul (Distributed Key-Value Store)**

**Why**:
- **Distributed Consensus**: Raft consensus for consistency
- **Service Registration**: Key-value store for service data
- **Watch API**: Real-time updates via watch mechanism
- **Health Checks**: Integrated health checking
- **Use Case**: Service registry, health checks, configuration

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Service metadata, configurations, relationships
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Services, instances, configurations, metadata

2. **Redis** (In-Memory Cache)
   - **Use Case**: Service registry cache, frequently accessed services
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Service cache (30 sec), health status (10 sec)

3. **ZooKeeper** (Coordination)
   - **Use Case**: Alternative service discovery, coordination
   - **Why**: Distributed coordination, consensus
   - **Schema**: Service nodes, health checks

4. **MongoDB** (Document Store)
   - **Use Case**: Service metadata, nested configurations
   - **Why**: Flexible schema, nested structures
   - **Schema**: Service documents with metadata, configurations

**Database Architecture**:
```
Service Registration → Service Discovery
    ├── etcd/Consul → Service registry, health checks, configuration
    ├── PostgreSQL → Service metadata, configurations
    ├── Redis → Service registry cache
    ├── ZooKeeper → Alternative coordination
    └── MongoDB → Service metadata, configurations
```

**Selection Matrix**:
| Requirement | etcd/Consul | PostgreSQL | Redis | ZooKeeper | MongoDB |
|------------|-------------|------------|-------|-----------|---------|
| Service Registration | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Health Checks | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ❌ Poor |
| Real-Time Updates | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ❌ Poor |
| Distributed Consensus | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Configuration Management | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |

---

### Problem 43: Design a Distributed Configuration System

#### Requirements
- **Scale**: 10K+ applications, 100K+ configurations, real-time updates
- **Features**: Configuration storage, versioning, change notifications, rollback
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Configurations, versions, metadata, change history
- **Consistency**: Strong consistency required (configuration)

#### Database Selection Criteria

**Primary Database: etcd/Consul (Distributed Key-Value Store)**

**Why**:
- **Distributed Consensus**: Strong consistency via Raft
- **Watch API**: Real-time configuration change notifications
- **Versioning**: Built-in versioning support
- **Low Latency**: Fast configuration reads
- **Use Case**: Configuration storage, versioning, notifications

**Secondary Databases**:

1. **Git** (Version Control)
   - **Use Case**: Configuration versioning, change history, rollback
   - **Why**: Version control, branching, merge capabilities
   - **Schema**: Configuration files, version history

2. **PostgreSQL** (Relational)
   - **Use Case**: Configuration metadata, relationships, audit trail
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Configurations, versions, metadata, audit logs

3. **Redis** (In-Memory Cache)
   - **Use Case**: Configuration cache, frequently accessed configs
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Configuration cache (5 min), invalidate on change

4. **MongoDB** (Document Store)
   - **Use Case**: Configuration documents, nested configurations
   - **Why**: Flexible schema, nested structures, JSON support
   - **Schema**: Configuration documents with nested values

5. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Configuration file storage, backups
   - **Why**: Cost-effective, versioning support
   - **Size**: Configuration files typically KBs

**Database Architecture**:
```
Configuration Request → Configuration Service
    ├── etcd/Consul → Configuration storage, versioning, notifications
    ├── Git → Configuration versioning, change history
    ├── PostgreSQL → Configuration metadata, audit trail
    ├── Redis → Configuration cache
    ├── MongoDB → Configuration documents
    └── S3 → Configuration file storage, backups
```

**Selection Matrix**:
| Requirement | etcd/Consul | Git | PostgreSQL | Redis | MongoDB |
|------------|-------------|-----|------------|-------|---------|
| Configuration Storage | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Versioning | ✅ Excellent | ✅ Excellent | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Real-Time Updates | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Change Notifications | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Rollback | ✅ Excellent | ✅ Excellent | ✅ Excellent | ❌ Poor | ⚠️ Moderate |

---

### Problem 44: Design a Distributed Lock Service

#### Requirements
- **Scale**: 1M+ locks, 10K+ lock operations/sec, distributed coordination
- **Features**: Lock acquisition, expiration, fairness, deadlock prevention
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Locks, ownership, expiration, metadata
- **Consistency**: Strong consistency required (locks)

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Atomic Operations**: SET NX EX for lock acquisition
- **Low Latency**: Sub-millisecond lock operations
- **TTL Support**: Automatic lock expiration
- **Lua Scripts**: Complex atomic operations
- **Use Case**: Distributed locks, leader election, coordination

**Secondary Databases**:

1. **ZooKeeper** (Coordination)
   - **Use Case**: Distributed locks, leader election, coordination
   - **Why**: Distributed consensus, ephemeral nodes
   - **Schema**: Lock nodes, leader nodes

2. **etcd** (Distributed Key-Value)
   - **Use Case**: Distributed locks, leader election
   - **Why**: Distributed consensus, TTL support, watch API
   - **Schema**: Lock keys with TTL

3. **PostgreSQL** (Relational)
   - **Use Case**: Lock metadata, audit trail, complex locking
   - **Why**: ACID transactions, advisory locks, complex queries
   - **Schema**: Lock records, metadata, audit logs

4. **Consul** (Service Discovery)
   - **Use Case**: Distributed locks, leader election
   - **Why**: Distributed consensus, session management
   - **Schema**: Lock keys, sessions

**Database Architecture**:
```
Lock Request → Lock Service
    ├── Redis → Distributed locks (SET NX EX)
    ├── ZooKeeper → Alternative locks, leader election
    ├── etcd → Alternative locks, coordination
    ├── PostgreSQL → Lock metadata, audit trail
    └── Consul → Alternative locks, sessions
```

**Selection Matrix**:
| Requirement | Redis | ZooKeeper | etcd | PostgreSQL | Consul |
|------------|-------|-----------|------|------------|--------|
| Lock Acquisition | ✅ Excellent | ✅ Excellent | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Automatic Expiration | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Distributed Consensus | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ❌ Poor | ✅ Excellent |
| Fairness | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |

---

### Problem 45: Design a Distributed Task Scheduler (Cron)

#### Requirements
- **Scale**: 1M+ scheduled tasks, 100K+ executions/day, distributed execution
- **Features**: Task scheduling, execution, retry, monitoring, timezone support
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Tasks, schedules, execution history, metadata
- **Consistency**: Strong consistency for scheduling

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for task scheduling, execution
- **Complex Queries**: Schedule queries, timezone handling
- **Relationships**: Tasks, schedules, executions, dependencies
- **Scheduling**: Efficient querying of due tasks
- **Use Case**: Task definitions, schedules, execution history

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Due tasks queue, execution queue, recent executions
   - **Why**: Sub-millisecond access, sorted sets for scheduling
   - **Data Structures**: Sorted sets (by execution time), lists (queues)

2. **Message Queue (RabbitMQ/Kafka)**
   - **Use Case**: Task execution queue, distributed execution
   - **Why**: Reliable delivery, multiple consumers, retry
   - **Pattern**: Producer-consumer for task execution

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Execution history, metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Execution history over time, metrics

4. **MongoDB** (Document Store)
   - **Use Case**: Task definitions, nested schedules, metadata
   - **Why**: Flexible schema, nested structures
   - **Schema**: Task documents with schedules, metadata

5. **ZooKeeper/etcd** (Coordination)
   - **Use Case**: Leader election, distributed coordination
   - **Why**: Distributed consensus, coordination
   - **Schema**: Leader nodes, coordination data

**Database Architecture**:
```
Task Schedule → Scheduler
    ├── PostgreSQL → Task definitions, schedules, execution history
    ├── Redis → Due tasks queue, execution queue
    ├── Message Queue → Task execution queue, distributed execution
    ├── TimescaleDB → Execution history, metrics
    ├── MongoDB → Task definitions, metadata
    └── ZooKeeper → Leader election, coordination
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | Message Queue | TimescaleDB | MongoDB |
|------------|------------|-------|---------------|-------------|---------|
| Task Scheduling | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Execution Queue | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor |
| Execution History | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent | ⚠️ Moderate |
| Retry Mechanism | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Distributed Execution | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ❌ Poor |

---

## Database Selection Framework (Continued)

### Infrastructure & DevOps Patterns

**Pattern 1: Service Discovery**
- **Requirement**: Distributed consensus, real-time updates
- **Databases**: etcd, Consul, ZooKeeper
- **Use Cases**: Microservices, service mesh, container orchestration

**Pattern 2: Configuration Management**
- **Requirement**: Versioning, change notifications, rollback
- **Databases**: etcd, Consul, Git, PostgreSQL
- **Use Cases**: Application configuration, feature flags

**Pattern 3: Distributed Coordination**
- **Requirement**: Locks, leader election, consensus
- **Databases**: Redis, ZooKeeper, etcd, Consul
- **Use Cases**: Distributed locks, leader election, coordination

**Pattern 4: Task Scheduling**
- **Requirement**: Scheduling, execution, retry, monitoring
- **Databases**: PostgreSQL, Redis, Message Queues
- **Use Cases**: Cron jobs, scheduled tasks, workflows

---

## Summary

**Problems Covered (41-45)**:
41. Load Balancer → Redis, PostgreSQL, etcd/Consul, TimescaleDB, Elasticsearch
42. Service Discovery → etcd/Consul, PostgreSQL, Redis, ZooKeeper, MongoDB
43. Distributed Configuration → etcd/Consul, Git, PostgreSQL, Redis, MongoDB
44. Distributed Lock Service → Redis, ZooKeeper, etcd, PostgreSQL, Consul
45. Distributed Task Scheduler → PostgreSQL, Redis, Message Queue, TimescaleDB, MongoDB

**Key Patterns Identified**:
- **Service Discovery**: etcd/Consul for distributed consensus
- **Configuration**: etcd/Consul or Git for versioning
- **Distributed Locks**: Redis or ZooKeeper for coordination
- **Task Scheduling**: PostgreSQL for scheduling, Redis/Queue for execution
- **Load Balancing**: Redis for session affinity and health status

**Next**: Problems 46-50 (Specialized Systems)

