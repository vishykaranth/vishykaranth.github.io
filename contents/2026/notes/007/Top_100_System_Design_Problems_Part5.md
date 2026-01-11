# Top 100 System Design Problems with Database Selection Criteria - Part 5

## Problems 21-25: Storage & File Systems

### Problem 21: Design a Distributed File System (Google File System)

#### Requirements
- **Scale**: Petabytes of data, millions of files, thousands of clients
- **Features**: File storage, replication, consistency, fault tolerance
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Files, metadata, chunks, blocks
- **Consistency**: Strong consistency for metadata, eventual for data

#### Database Selection Criteria

**Primary Database: Custom Distributed File System**

**Why**:
- **Large Files**: Optimized for large file storage
- **Chunking**: Files split into chunks for distribution
- **Replication**: Multiple replicas for fault tolerance
- **Metadata**: Separate metadata server (can use database)
- **Use Case**: File storage, chunk management, replication

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: File metadata, namespace, permissions
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Files, directories, permissions, metadata

2. **Redis** (In-Memory Cache)
   - **Use Case**: Metadata cache, directory cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Metadata cache (5 min), directory cache (1 min)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: File metadata (alternative), chunk locations
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `files_by_directory`, `chunks_by_file`

4. **HDFS/Distributed Storage**
   - **Use Case**: Actual file chunks, data storage
   - **Why**: Distributed storage, fault tolerance
   - **Size**: Chunks typically 64MB-256MB

5. **ZooKeeper/etcd** (Coordination)
   - **Use Case**: Leader election, configuration, locks
   - **Why**: Distributed coordination, consensus
   - **Use Case**: Metadata server coordination

**Database Architecture**:
```
File Operation → Client
    ├── Metadata Server (PostgreSQL) → File metadata, namespace
    ├── Redis → Metadata cache
    ├── Cassandra → Chunk locations (alternative)
    ├── HDFS → File chunks, data storage
    └── ZooKeeper → Coordination, leader election
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | Cassandra | HDFS | ZooKeeper |
|------------|------------|-------|-----------|------|-----------|
| Metadata Storage | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| File Storage | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Metadata Cache | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Coordination | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |
| Fault Tolerance | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ✅ Excellent |

---

### Problem 22: Design a Cloud Storage Service (Dropbox/Google Drive)

#### Requirements
- **Scale**: 1B+ users, 1PB+ storage, 1B+ files
- **Features**: File upload/download, sync, versioning, sharing
- **Read/Write Ratio**: 5:1 (reads dominate)
- **Data Types**: Files, metadata, versions, sharing, sync state
- **Consistency**: Strong consistency for metadata, eventual for sync

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for file operations, sharing
- **Complex Queries**: File relationships, sharing, permissions
- **Versioning**: File version management
- **Schema**: Normalized relational schema
- **Use Case**: File metadata, users, sharing, permissions

**Secondary Databases**:

1. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Actual file storage, versions
   - **Why**: Cost-effective, scalable, versioning support
   - **Size**: Files from KBs to GBs, multiple versions

2. **Redis** (In-Memory Cache)
   - **Use Case**: Sync state, recent files cache, session storage
   - **Why**: Sub-millisecond latency, pub/sub for real-time sync
   - **TTL**: Sync state (1 hour), recent files (30 min)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: File metadata (alternative), sync logs
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `files_by_user`, `versions_by_file`

4. **MongoDB** (Document Store)
   - **Use Case**: File metadata, nested folder structures
   - **Why**: Flexible schema, nested structures
   - **Schema**: File documents with metadata, versions

5. **Elasticsearch** (Search Engine)
   - **Use Case**: File search, content search
   - **Why**: Full-text search, relevance ranking
   - **Schema**: File documents with content, metadata

**Database Architecture**:
```
File Operation → API Gateway
    ├── PostgreSQL → File metadata, users, sharing, permissions
    ├── S3 → File storage, versions
    ├── Redis → Sync state, recent files cache
    ├── Cassandra → File metadata (alternative), sync logs
    ├── MongoDB → File metadata, folder structures
    └── Elasticsearch → File search
```

**Selection Matrix**:
| Requirement | PostgreSQL | S3 | Redis | Cassandra | MongoDB |
|------------|------------|----|----|-----------|---------|
| File Storage | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| File Metadata | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |
| Sync State | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |
| File Sharing | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |
| File Search | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate |

---

### Problem 23: Design a Distributed Cache (Redis Cluster)

#### Requirements
- **Scale**: 1B+ keys, 1M+ ops/sec, terabytes of data
- **Features**: Sharding, replication, consistency, eviction
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Key-value pairs, various data structures
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis Cluster (In-Memory Cache)**

**Why**:
- **In-Memory**: Sub-millisecond latency
- **Data Structures**: Strings, hashes, lists, sets, sorted sets
- **Sharding**: Consistent hashing for distribution
- **Replication**: Master-slave replication
- **Use Case**: Caching, session storage, real-time data

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Persistent storage, cache source data
   - **Why**: ACID transactions, complex queries
   - **Schema**: Source data that gets cached

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Persistent cache backup, large-scale cache
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: Cache data by key, time-based expiration

3. **Memcached** (In-Memory Cache)
   - **Use Case**: Simple key-value caching (alternative)
   - **Why**: Simpler than Redis, good for basic caching
   - **Limitation**: No persistence, no data structures

4. **DynamoDB** (NoSQL - Key-Value)
   - **Use Case**: Managed cache alternative
   - **Why**: Managed service, auto-scaling, low latency
   - **Schema**: Key-value pairs with TTL

**Database Architecture**:
```
Cache Request → Load Balancer
    ├── Redis Cluster → Distributed cache (sharded, replicated)
    ├── PostgreSQL → Source data (cache miss)
    ├── Cassandra → Persistent cache backup
    └── Memcached → Alternative cache layer
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | Memcached | DynamoDB |
|------------|-------|------------|-----------|-----------|----------|
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ✅ Good |
| Data Structures | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Persistence | ✅ Good | ✅ Excellent | ✅ Excellent | ❌ Poor | ✅ Excellent |
| Sharding | ✅ Excellent | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Replication | ✅ Excellent | ✅ Excellent | ✅ Excellent | ❌ Poor | ✅ Excellent |

---

### Problem 24: Design a Key-Value Store (DynamoDB)

#### Requirements
- **Scale**: 1B+ items, 1M+ ops/sec, petabytes of data
- **Features**: Partitioning, replication, consistency, availability
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Key-value pairs, documents
- **Consistency**: Configurable (strong or eventual)

#### Database Selection Criteria

**Primary Database: DynamoDB (NoSQL - Key-Value)**

**Why**:
- **Managed Service**: No infrastructure management
- **Auto-Scaling**: Automatic scaling based on load
- **Low Latency**: Single-digit millisecond latency
- **Consistency**: Configurable consistency levels
- **Use Case**: Key-value storage, session data, user data

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Hot data cache, frequently accessed items
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Hot data cache (5-15 min)

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Alternative key-value store, large-scale
   - **Why**: High throughput, horizontal scaling
   - **Schema**: Key-value pairs, wide columns

3. **PostgreSQL** (Relational)
   - **Use Case**: Source data, complex queries
   - **Why**: ACID transactions, complex queries
   - **Schema**: Normalized data that gets key-value accessed

4. **MongoDB** (Document Store)
   - **Use Case**: Document storage, nested structures
   - **Why**: Flexible schema, document model
   - **Schema**: Documents with nested key-value pairs

**Database Architecture**:
```
Key-Value Request → API Gateway
    ├── DynamoDB → Key-value storage (partitioned, replicated)
    ├── Redis → Hot data cache
    ├── Cassandra → Alternative key-value store
    ├── PostgreSQL → Source data
    └── MongoDB → Document storage
```

**Selection Matrix**:
| Requirement | DynamoDB | Redis | Cassandra | PostgreSQL | MongoDB |
|------------|----------|-------|-----------|------------|---------|
| Key-Value Storage | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |
| Auto-Scaling | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Low Latency | ✅ Good | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Consistency Options | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Managed Service | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |

---

### Problem 25: Design a Distributed Database (Cassandra)

#### Requirements
- **Scale**: Petabytes of data, millions of ops/sec, thousands of nodes
- **Features**: Partitioning, replication, consistency, write/read paths
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Wide columns, time-series, key-value
- **Consistency**: Eventual consistency (tunable)

#### Database Selection Criteria

**Primary Database: Cassandra (NoSQL - Wide Column)**

**Why**:
- **Horizontal Scaling**: Linear scaling with nodes
- **High Write Throughput**: Optimized for writes
- **Partitioning**: Consistent hashing for distribution
- **Replication**: Multi-datacenter replication
- **Use Case**: Time-series data, high-volume writes, distributed storage

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Metadata, configuration, reference data
   - **Why**: ACID transactions, complex queries
   - **Schema**: Metadata, configurations

2. **Redis** (In-Memory Cache)
   - **Use Case**: Hot data cache, frequently accessed rows
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Hot data cache (5-15 min)

3. **Elasticsearch** (Search Engine)
   - **Use Case**: Search over Cassandra data
   - **Why**: Full-text search, aggregations
   - **Schema**: Indexed Cassandra data

4. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Backup, snapshots, archival
   - **Why**: Cost-effective, durable storage
   - **Size**: Snapshots can be large

5. **Kafka** (Message Queue)
   - **Use Case**: Data ingestion, change data capture
   - **Why**: High throughput, reliable delivery
   - **Pattern**: Producer-consumer for data ingestion

**Database Architecture**:
```
Data Request → API Gateway
    ├── Cassandra Cluster → Distributed database (partitioned, replicated)
    ├── PostgreSQL → Metadata, configuration
    ├── Redis → Hot data cache
    ├── Elasticsearch → Search index
    ├── S3 → Backup, snapshots
    └── Kafka → Data ingestion
```

**Selection Matrix**:
| Requirement | Cassandra | PostgreSQL | Redis | Elasticsearch | S3 |
|------------|-----------|------------|-------|---------------|----|
| Horizontal Scaling | ✅ Excellent | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| Write Throughput | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Good |
| Partitioning | ✅ Excellent | ❌ Poor | ✅ Excellent | ✅ Excellent | N/A |
| Replication | ✅ Excellent | ✅ Excellent | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| Backup Storage | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent |

---

## Database Selection Framework (Continued)

### Storage & File System Patterns

**Pattern 1: File Storage**
- **Requirement**: Large files, versioning
- **Databases**: S3, Azure Blob, Google Cloud Storage, HDFS
- **Use Cases**: Cloud storage, file systems, backups

**Pattern 2: Metadata Management**
- **Requirement**: File metadata, namespace, permissions
- **Databases**: PostgreSQL, MongoDB, Cassandra
- **Use Cases**: File systems, cloud storage, document management

**Pattern 3: Distributed Caching**
- **Requirement**: Low latency, high throughput
- **Databases**: Redis Cluster, Memcached, DynamoDB
- **Use Cases**: Application caching, session storage

**Pattern 4: Key-Value Storage**
- **Requirement**: Simple key-value access, high scale
- **Databases**: DynamoDB, Redis, Cassandra, Riak
- **Use Cases**: Session storage, user data, configuration

---

## Summary

**Problems Covered (21-25)**:
21. Distributed File System → PostgreSQL, Redis, Cassandra, HDFS, ZooKeeper
22. Cloud Storage Service → PostgreSQL, S3, Redis, Cassandra, MongoDB
23. Distributed Cache → Redis, PostgreSQL, Cassandra, Memcached, DynamoDB
24. Key-Value Store → DynamoDB, Redis, Cassandra, PostgreSQL, MongoDB
25. Distributed Database → Cassandra, PostgreSQL, Redis, Elasticsearch, S3

**Key Patterns Identified**:
- **File Storage**: S3/Blob storage for large files
- **Metadata**: PostgreSQL for complex metadata queries
- **Caching**: Redis for low-latency caching
- **Key-Value**: DynamoDB or Redis for simple key-value access
- **Distributed Storage**: Cassandra for high-scale distributed storage

**Next**: Problems 26-30 (Analytics & Monitoring Systems)

