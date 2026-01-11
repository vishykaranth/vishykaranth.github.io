# Top 100 System Design Problems with Database Selection Criteria - Part 16

## Problems 76-80: Additional Analytics Systems

### Problem 76: Design a Business Intelligence System

#### Requirements
- **Scale**: Petabytes of data, millions of queries/day, complex analytics
- **Features**: OLAP queries, data warehousing, reporting, dashboards, ETL
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Dimensions, facts, aggregations, reports, dashboards
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Data Warehouse (Snowflake/Redshift/BigQuery)**

**Why**:
- **OLAP Queries**: Optimized for analytical queries
- **Columnar Storage**: Efficient for aggregations and analytics
- **Scalability**: Auto-scaling for large queries
- **ETL Integration**: Built-in ETL capabilities
- **Use Case**: Data warehousing, OLAP queries, analytics, reporting

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Metadata, configurations, relationships, reference data
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Metadata, configurations, reference data

2. **Redis** (In-Memory Cache)
   - **Use Case**: Report cache, dashboard cache, frequently accessed data
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Report cache (1 hour), dashboards (30 min)

3. **Elasticsearch** (Search Engine)
   - **Use Case**: Report search, dashboard search, content discovery
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Report documents, dashboard documents

4. **TimescaleDB** (Time-Series)
   - **Use Case**: Time-series analytics, metrics, trends
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Metrics over time, trends, analytics

5. **ClickHouse** (Analytical Database)
   - **Use Case**: OLAP queries, aggregations, analytics (alternative)
   - **Why**: Columnar storage, fast aggregations
   - **Schema**: Facts, dimensions, aggregations

**Database Architecture**:
```
Analytics Query → BI Service
    ├── Data Warehouse → OLAP queries, data warehousing
    ├── PostgreSQL → Metadata, configurations, reference data
    ├── Redis → Report cache, dashboard cache
    ├── Elasticsearch → Report search, dashboard search
    ├── TimescaleDB → Time-series analytics, metrics
    └── ClickHouse → OLAP queries (alternative), aggregations
```

**Selection Matrix**:
| Requirement | Data Warehouse | PostgreSQL | Redis | Elasticsearch | ClickHouse |
|------------|---------------|------------|-------|---------------|------------|
| OLAP Queries | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent |
| Data Warehousing | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| Report Generation | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Report Search | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ❌ Poor |
| Time-Series Analytics | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent |

---

### Problem 77: Design a Data Pipeline System

#### Requirements
- **Scale**: 1B+ records/day, real-time and batch processing, multiple sources
- **Features**: Data ingestion, transformation, validation, loading, monitoring
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Raw data, transformed data, metadata, pipeline state
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Kafka (Message Queue)**

**Why**:
- **Data Streaming**: Real-time data ingestion and streaming
- **High Throughput**: Millions of records per second
- **Partitioning**: Distributed processing across partitions
- **Replication**: Fault-tolerant data streaming
- **Use Case**: Data ingestion, streaming, event processing

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Pipeline metadata, configurations, state management
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Pipelines, configurations, state, metadata

2. **TimescaleDB** (Time-Series)
   - **Use Case**: Pipeline metrics, processing metrics, performance data
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Pipeline metrics over time, processing metrics

3. **Redis** (In-Memory Cache)
   - **Use Case**: Pipeline state cache, recent data cache, job status
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: State cache (1 hour), job status (24 hours)

4. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Raw data storage, processed data storage, staging
   - **Why**: Cost-effective, scalable storage
   - **Size**: Data files can be large

5. **Elasticsearch** (Search Engine)
   - **Use Case**: Data search, metadata search, pipeline search
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Data documents, pipeline documents, metadata

**Database Architecture**:
```
Data Source → Data Pipeline
    ├── Kafka → Data ingestion, streaming, event processing
    ├── PostgreSQL → Pipeline metadata, configurations, state
    ├── TimescaleDB → Pipeline metrics, processing metrics
    ├── Redis → Pipeline state cache, job status
    ├── S3 → Raw data storage, processed data storage
    └── Elasticsearch → Data search, metadata search
```

**Selection Matrix**:
| Requirement | Kafka | PostgreSQL | TimescaleDB | Redis | S3 |
|------------|-------|------------|-------------|-------|----|
| Data Streaming | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Pipeline Management | ❌ Poor | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Pipeline Metrics | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor |
| Data Storage | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |
| High Throughput | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ✅ Good |

---

### Problem 78: Design an Event Sourcing System

#### Requirements
- **Scale**: 1B+ events/day, event history, snapshots, replay
- **Features**: Event storage, event replay, snapshots, projections
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Events, snapshots, projections, metadata
- **Consistency**: Strong consistency for events

#### Database Selection Criteria

**Primary Database: Event Store (EventStore/Kafka)**

**Why**:
- **Event Storage**: Optimized for event storage and retrieval
- **Event Replay**: Efficient event replay capabilities
- **Append-Only**: Append-only event log
- **Streaming**: Event streaming support
- **Use Case**: Event storage, event replay, event sourcing

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Snapshots, projections, metadata, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Snapshots, projections, metadata, configurations

2. **Redis** (In-Memory Cache)
   - **Use Case**: Recent events cache, snapshots cache, projections cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Recent events (1 hour), snapshots (24 hours)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Event storage (alternative), event history
   - **Why**: High write throughput, time-series data
   - **Schema**: `events_by_stream`, `events_by_timestamp`

4. **MongoDB** (Document Store)
   - **Use Case**: Projections, snapshots, nested event data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Projection documents, snapshot documents

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Event metrics, replay metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Event metrics over time, replay metrics

**Database Architecture**:
```
Event → Event Store
    ├── Event Store → Event storage, event replay
    ├── PostgreSQL → Snapshots, projections, metadata
    ├── Redis → Recent events cache, snapshots cache
    ├── Cassandra → Event storage (alternative), history
    ├── MongoDB → Projections, snapshots, nested data
    └── TimescaleDB → Event metrics, replay metrics
```

**Selection Matrix**:
| Requirement | Event Store | PostgreSQL | Redis | Cassandra | MongoDB |
|------------|-------------|------------|-------|-----------|---------|
| Event Storage | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate |
| Event Replay | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Snapshots | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Projections | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Event Metrics | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |

---

### Problem 79: Design a CQRS System

#### Requirements
- **Scale**: 1B+ commands/day, 10B+ reads/day, separate read/write models
- **Features**: Command handling, query handling, read models, write models
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Commands, events, read models, write models, projections
- **Consistency**: Strong for writes, eventual for reads

#### Database Selection Criteria

**Write Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for command handling
- **Strong Consistency**: Required for write operations
- **Complex Queries**: Command validation, business logic
- **Relationships**: Command relationships, write models
- **Use Case**: Command storage, write models, business logic

**Read Database: Elasticsearch/Redis (Optimized for Reads)**

**Why**:
- **Read Optimization**: Optimized for read operations
- **High Throughput**: Millions of reads per second
- **Low Latency**: Sub-millisecond read latency
- **Scalability**: Horizontal scaling for reads
- **Use Case**: Read models, query handling, projections

**Secondary Databases**:

1. **Event Store (Kafka/EventStore)**
   - **Use Case**: Event storage, event streaming, projections
   - **Why**: Event sourcing, event streaming, projections
   - **Schema**: Events, streams, projections

2. **Redis** (In-Memory Cache)
   - **Use Case**: Read model cache, frequently accessed data
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Read model cache (1 hour), frequently accessed (24 hours)

3. **MongoDB** (Document Store)
   - **Use Case**: Read models, projections, nested data
   - **Why**: Flexible schema, nested structures, read optimization
   - **Schema**: Read model documents, projection documents

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Read models (alternative), high-scale reads
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `read_models_by_query`, `read_models_by_user`

**Database Architecture**:
```
Command → Write Side → Event Store → Read Side
    ├── PostgreSQL → Command storage, write models (ACID)
    ├── Event Store → Event storage, event streaming
    ├── Elasticsearch/Redis → Read models, query handling
    ├── Redis → Read model cache, frequently accessed
    ├── MongoDB → Read models, projections, nested data
    └── Cassandra → Read models (alternative), high-scale
```

**Selection Matrix**:
| Requirement | PostgreSQL | Event Store | Elasticsearch/Redis | MongoDB | Cassandra |
|------------|------------|-------------|---------------------|---------|-----------|
| Write Operations | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Read Operations | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| Event Storage | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Read Models | ❌ Poor | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| Projections | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |

---

### Problem 80: Design a Data Warehouse

#### Requirements
- **Scale**: Petabytes of data, billions of rows, complex analytics
- **Features**: ETL, dimensional modeling, OLAP queries, reporting
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Facts, dimensions, aggregations, reports, metadata
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Data Warehouse (Snowflake/Redshift/BigQuery)**

**Why**:
- **OLAP Queries**: Optimized for analytical queries
- **Columnar Storage**: Efficient for aggregations and analytics
- **Scalability**: Auto-scaling for large queries
- **ETL Integration**: Built-in ETL capabilities
- **Use Case**: Data warehousing, OLAP queries, analytics, reporting

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Metadata, configurations, relationships, reference data
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Metadata, configurations, reference data

2. **Redis** (In-Memory Cache)
   - **Use Case**: Report cache, dashboard cache, frequently accessed data
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Report cache (1 hour), dashboards (30 min)

3. **Elasticsearch** (Search Engine)
   - **Use Case**: Report search, dashboard search, content discovery
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Report documents, dashboard documents

4. **TimescaleDB** (Time-Series)
   - **Use Case**: Time-series analytics, metrics, trends
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Metrics over time, trends, analytics

5. **ClickHouse** (Analytical Database)
   - **Use Case**: OLAP queries, aggregations, analytics (alternative)
   - **Why**: Columnar storage, fast aggregations
   - **Schema**: Facts, dimensions, aggregations

**Database Architecture**:
```
ETL → Data Warehouse
    ├── Data Warehouse → OLAP queries, data warehousing
    ├── PostgreSQL → Metadata, configurations, reference data
    ├── Redis → Report cache, dashboard cache
    ├── Elasticsearch → Report search, dashboard search
    ├── TimescaleDB → Time-series analytics, metrics
    └── ClickHouse → OLAP queries (alternative), aggregations
```

**Selection Matrix**:
| Requirement | Data Warehouse | PostgreSQL | Redis | Elasticsearch | ClickHouse |
|------------|---------------|------------|-------|---------------|------------|
| OLAP Queries | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent |
| Data Warehousing | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| Report Generation | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Report Search | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ❌ Poor |
| Time-Series Analytics | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent |

---

## Database Selection Framework (Continued)

### Additional Analytics System Patterns

**Pattern 1: Business Intelligence**
- **Requirement**: OLAP queries, data warehousing, reporting
- **Databases**: Snowflake, Redshift, BigQuery, ClickHouse
- **Use Cases**: BI dashboards, reporting, analytics

**Pattern 2: Data Pipelines**
- **Requirement**: Data ingestion, transformation, streaming
- **Databases**: Kafka, PostgreSQL, S3, TimescaleDB
- **Use Cases**: ETL, data processing, real-time analytics

**Pattern 3: Event Sourcing**
- **Requirement**: Event storage, replay, snapshots, projections
- **Databases**: EventStore, Kafka, PostgreSQL, MongoDB
- **Use Cases**: Event sourcing, CQRS, audit trails

**Pattern 4: CQRS**
- **Requirement**: Separate read/write models, eventual consistency
- **Databases**: PostgreSQL (write), Elasticsearch/Redis (read), Event Store
- **Use Cases**: High-read systems, read optimization, scalability

---

## Summary

**Problems Covered (76-80)**:
76. Business Intelligence System → Data Warehouse, PostgreSQL, Redis, Elasticsearch, ClickHouse
77. Data Pipeline System → Kafka, PostgreSQL, TimescaleDB, Redis, S3
78. Event Sourcing System → Event Store, PostgreSQL, Redis, Cassandra, MongoDB
79. CQRS System → PostgreSQL, Event Store, Elasticsearch/Redis, MongoDB, Cassandra
80. Data Warehouse → Data Warehouse, PostgreSQL, Redis, Elasticsearch, ClickHouse

**Key Patterns Identified**:
- **BI Systems**: Data warehouses for OLAP queries and analytics
- **Data Pipelines**: Kafka for streaming, S3 for storage
- **Event Sourcing**: Event stores for event storage and replay
- **CQRS**: Separate databases for reads and writes
- **Data Warehousing**: Columnar databases for analytical queries

**Next**: Problems 81-85 (Additional Real-Time Systems)

