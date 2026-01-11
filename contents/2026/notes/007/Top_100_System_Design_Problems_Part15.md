# Top 100 System Design Problems with Database Selection Criteria - Part 15

## Problems 71-75: Additional Storage Systems

### Problem 71: Design an Object Storage Service

#### Requirements
- **Scale**: Petabytes of data, billions of objects, unlimited scale
- **Features**: Object storage, versioning, lifecycle policies, access control
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Objects, metadata, versions, access policies
- **Consistency**: Strong consistency for metadata, eventual for data

#### Database Selection Criteria

**Primary Database: S3/Blob Storage (Object Storage)**

**Why**:
- **Object Storage**: Native object storage architecture
- **Unlimited Scale**: Virtually unlimited storage capacity
- **Cost-Effective**: Low-cost storage for large volumes
- **Versioning**: Built-in versioning support
- **Use Case**: Object storage, file storage, backup, archival

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Object metadata, access policies, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Objects, metadata, policies, configurations

2. **Redis** (In-Memory Cache)
   - **Use Case**: Object metadata cache, frequently accessed objects
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Metadata cache (1 hour), objects (30 min)

3. **Elasticsearch** (Search Engine)
   - **Use Case**: Object search, metadata search, content discovery
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Object documents with metadata, content

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Object metadata (alternative), high-scale storage
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `objects_by_bucket`, `objects_by_timestamp`

5. **MongoDB** (Document Store)
   - **Use Case**: Object metadata, nested metadata, flexible schema
   - **Why**: Flexible schema, nested structures
   - **Schema**: Object documents with metadata, versions

**Database Architecture**:
```
Object Operation → Object Storage Service
    ├── S3/Blob Storage → Object storage, versions
    ├── PostgreSQL → Object metadata, access policies
    ├── Redis → Object metadata cache, frequently accessed
    ├── Elasticsearch → Object search, metadata search
    ├── Cassandra → Object metadata (alternative)
    └── MongoDB → Object metadata, nested data
```

**Selection Matrix**:
| Requirement | S3 | PostgreSQL | Redis | Elasticsearch | Cassandra |
|------------|----|------------|-------|---------------|-----------|
| Object Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Object Metadata | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| Object Search | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ❌ Poor |
| Versioning | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Access Policies | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |

---

### Problem 72: Design a Block Storage Service

#### Requirements
- **Scale**: Petabytes of data, millions of volumes, high IOPS
- **Features**: Block storage, snapshots, cloning, encryption, performance tiers
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Blocks, volumes, snapshots, metadata, performance metrics
- **Consistency**: Strong consistency required

#### Database Selection Criteria

**Primary Database: Block Storage (EBS/Azure Disk)**

**Why**:
- **Block Storage**: Native block storage architecture
- **High IOPS**: High input/output operations per second
- **Snapshots**: Built-in snapshot support
- **Performance Tiers**: Multiple performance tiers
- **Use Case**: Block storage, volumes, snapshots, high-performance storage

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Volume metadata, snapshot metadata, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Volumes, snapshots, metadata, configurations

2. **Redis** (In-Memory Cache)
   - **Use Case**: Volume metadata cache, frequently accessed volumes
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Metadata cache (1 hour), volumes (30 min)

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Volume metrics, IOPS metrics, performance data
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Volume metrics over time, IOPS data

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Volume metadata (alternative), high-scale storage
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `volumes_by_region`, `snapshots_by_volume`

5. **MongoDB** (Document Store)
   - **Use Case**: Volume metadata, nested metadata, flexible schema
   - **Why**: Flexible schema, nested structures
   - **Schema**: Volume documents with metadata, snapshots

**Database Architecture**:
```
Volume Operation → Block Storage Service
    ├── Block Storage → Volumes, blocks, snapshots
    ├── PostgreSQL → Volume metadata, snapshot metadata
    ├── Redis → Volume metadata cache, frequently accessed
    ├── TimescaleDB → Volume metrics, IOPS metrics
    ├── Cassandra → Volume metadata (alternative)
    └── MongoDB → Volume metadata, nested data
```

**Selection Matrix**:
| Requirement | Block Storage | PostgreSQL | Redis | TimescaleDB | Cassandra |
|------------|---------------|------------|-------|-------------|-----------|
| Block Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| High IOPS | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Snapshots | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Volume Metadata | ❌ Poor | ✅ Excellent | ✅ Excellent | ❌ Poor | ✅ Excellent |
| Performance Metrics | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |

---

### Problem 73: Design a Backup System

#### Requirements
- **Scale**: Petabytes of backups, millions of backup jobs, long-term retention
- **Features**: Backup scheduling, incremental backups, restore, compression, encryption
- **Read/Write Ratio**: 1:10 (writes dominate)
- **Data Types**: Backups, metadata, schedules, restore points, retention policies
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: S3/Blob Storage (Object Storage)**

**Why**:
- **Backup Storage**: Cost-effective storage for backups
- **Unlimited Scale**: Virtually unlimited storage capacity
- **Lifecycle Policies**: Automated lifecycle management
- **Durability**: High durability for backup data
- **Use Case**: Backup storage, long-term retention, archival

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Backup metadata, schedules, restore points, policies
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Backups, schedules, restore points, policies

2. **Redis** (In-Memory Cache)
   - **Use Case**: Backup job status, recent backups cache
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Job status (24 hours), recent backups (1 hour)

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Backup metrics, restore metrics, performance data
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Backup metrics over time, restore metrics

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Backup metadata (alternative), backup history
   - **Why**: High write throughput, time-series data
   - **Schema**: `backups_by_source`, `backups_by_timestamp`

5. **MongoDB** (Document Store)
   - **Use Case**: Backup metadata, nested backup data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Backup documents with metadata, restore points

**Database Architecture**:
```
Backup Job → Backup Service
    ├── S3 → Backup storage, long-term retention
    ├── PostgreSQL → Backup metadata, schedules, policies
    ├── Redis → Backup job status, recent backups cache
    ├── TimescaleDB → Backup metrics, restore metrics
    ├── Cassandra → Backup metadata (alternative), history
    └── MongoDB → Backup metadata, nested data
```

**Selection Matrix**:
| Requirement | S3 | PostgreSQL | Redis | TimescaleDB | Cassandra |
|------------|----|------------|-------|-------------|-----------|
| Backup Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Backup Scheduling | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Backup Metadata | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent |
| Backup Metrics | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ⚠️ Moderate |
| Long-Term Retention | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |

---

### Problem 74: Design an Archive System

#### Requirements
- **Scale**: Petabytes of archived data, long-term retention, cost optimization
- **Features**: Archival, retrieval, compression, encryption, lifecycle management
- **Read/Write Ratio**: 1:100 (writes dominate, rare reads)
- **Data Types**: Archived data, metadata, retrieval requests, policies
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: S3 Glacier/Cold Storage (Object Storage)**

**Why**:
- **Cost-Effective**: Lowest cost storage for archival
- **Long-Term Retention**: Optimized for long-term storage
- **Lifecycle Management**: Automated lifecycle transitions
- **Durability**: High durability for archived data
- **Use Case**: Long-term archival, cold storage, compliance storage

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Archive metadata, retrieval requests, policies
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Archives, retrieval requests, policies, metadata

2. **Redis** (In-Memory Cache)
   - **Use Case**: Retrieval request status, recent retrievals cache
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Request status (24 hours), recent retrievals (1 hour)

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Archive metrics, retrieval metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Archive metrics over time, retrieval metrics

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Archive metadata (alternative), archive history
   - **Why**: High write throughput, time-series data
   - **Schema**: `archives_by_source`, `archives_by_timestamp`

5. **MongoDB** (Document Store)
   - **Use Case**: Archive metadata, nested archive data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Archive documents with metadata, retrieval info

**Database Architecture**:
```
Archive Request → Archive Service
    ├── S3 Glacier → Archive storage, cold storage
    ├── PostgreSQL → Archive metadata, retrieval requests, policies
    ├── Redis → Retrieval request status, recent retrievals
    ├── TimescaleDB → Archive metrics, retrieval metrics
    ├── Cassandra → Archive metadata (alternative), history
    └── MongoDB → Archive metadata, nested data
```

**Selection Matrix**:
| Requirement | S3 Glacier | PostgreSQL | Redis | TimescaleDB | Cassandra |
|------------|------------|------------|-------|-------------|-----------|
| Archive Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Cost Optimization | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Archive Metadata | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent |
| Retrieval Requests | ❌ Poor | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor |
| Archive Metrics | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ⚠️ Moderate |

---

### Problem 75: Design a Data Lake

#### Requirements
- **Scale**: Exabytes of data, billions of files, multiple data sources
- **Features**: Data ingestion, storage, cataloging, querying, analytics
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Raw data, processed data, metadata, schemas, catalogs
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: S3/HDFS (Distributed File System)**

**Why**:
- **Data Storage**: Cost-effective storage for large-scale data
- **Unlimited Scale**: Virtually unlimited storage capacity
- **Multiple Formats**: Support for various data formats
- **Distributed**: Distributed storage architecture
- **Use Case**: Raw data storage, processed data, data lake storage

**Secondary Databases**:

1. **Data Catalog (Hive Metastore/Glue)**
   - **Use Case**: Data catalog, schema registry, metadata management
   - **Why**: Centralized metadata, schema management
   - **Schema**: Tables, schemas, partitions, metadata

2. **PostgreSQL** (Relational)
   - **Use Case**: Data catalog metadata, relationships, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Data catalog, schemas, relationships, configurations

3. **Elasticsearch** (Search Engine)
   - **Use Case**: Data search, metadata search, content discovery
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Data documents with metadata, schemas

4. **TimescaleDB** (Time-Series)
   - **Use Case**: Data ingestion metrics, query metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Ingestion metrics over time, query metrics

5. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Data metadata (alternative), high-scale storage
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `data_by_source`, `data_by_timestamp`

**Database Architecture**:
```
Data Ingestion → Data Lake
    ├── S3/HDFS → Data storage, raw data, processed data
    ├── Data Catalog → Schema registry, metadata management
    ├── PostgreSQL → Data catalog metadata, relationships
    ├── Elasticsearch → Data search, metadata search
    ├── TimescaleDB → Data ingestion metrics, query metrics
    └── Cassandra → Data metadata (alternative)
```

**Selection Matrix**:
| Requirement | S3/HDFS | Data Catalog | PostgreSQL | Elasticsearch | TimescaleDB |
|------------|---------|--------------|------------|---------------|-------------|
| Data Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Data Catalog | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ❌ Poor |
| Data Search | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ❌ Poor |
| Schema Management | ❌ Poor | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor |
| Data Analytics | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |

---

## Database Selection Framework (Continued)

### Additional Storage System Patterns

**Pattern 1: Object Storage**
- **Requirement**: Unlimited scale, cost-effective, versioning
- **Databases**: S3, Azure Blob, Google Cloud Storage
- **Use Cases**: File storage, backup, archival, media storage

**Pattern 2: Block Storage**
- **Requirement**: High IOPS, snapshots, performance tiers
- **Databases**: EBS, Azure Disk, Google Persistent Disk
- **Use Cases**: Database storage, high-performance storage, volumes

**Pattern 3: Backup & Archive**
- **Requirement**: Cost-effective, long-term retention, lifecycle management
- **Databases**: S3 Glacier, Azure Archive, Google Coldline
- **Use Cases**: Backup, archival, compliance storage, long-term retention

**Pattern 4: Data Lake**
- **Requirement**: Large-scale storage, multiple formats, cataloging
- **Databases**: S3, HDFS, Data Catalogs (Hive, Glue)
- **Use Cases**: Data lakes, analytics, big data storage

---

## Summary

**Problems Covered (71-75)**:
71. Object Storage Service → S3, PostgreSQL, Redis, Elasticsearch, Cassandra
72. Block Storage Service → Block Storage, PostgreSQL, Redis, TimescaleDB, Cassandra
73. Backup System → S3, PostgreSQL, Redis, TimescaleDB, Cassandra
74. Archive System → S3 Glacier, PostgreSQL, Redis, TimescaleDB, Cassandra
75. Data Lake → S3/HDFS, Data Catalog, PostgreSQL, Elasticsearch, TimescaleDB

**Key Patterns Identified**:
- **Object Storage**: S3 for unlimited scale and cost-effectiveness
- **Block Storage**: EBS/Azure Disk for high IOPS and performance
- **Backup/Archive**: S3 Glacier for cost-effective long-term storage
- **Data Lake**: S3/HDFS for large-scale data storage with cataloging
- **Metadata Management**: PostgreSQL for complex metadata queries

**Next**: Problems 76-80 (Additional Analytics Systems)

