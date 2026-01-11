# Top 100 System Design Problems with Database Selection Criteria - Part 6

## Problems 26-30: Analytics & Monitoring Systems

### Problem 26: Design a Distributed Logging System

#### Requirements
- **Scale**: 1B+ log entries/day, 100K+ logs/sec, petabytes of data
- **Features**: Log aggregation, storage, querying, real-time processing
- **Read/Write Ratio**: 1:10 (writes dominate)
- **Data Types**: Log entries, metadata, timestamps, structured/unstructured
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Elasticsearch (Search Engine)**

**Why**:
- **Full-Text Search**: Search across log content
- **Time-Series**: Efficient time-based queries
- **Aggregations**: Log analytics, aggregations
- **Real-Time**: Near real-time indexing
- **Use Case**: Log storage, search, analytics

**Secondary Databases**:

1. **Kafka** (Message Queue)
   - **Use Case**: Log ingestion pipeline, buffering
   - **Why**: High throughput, reliable delivery, streaming
   - **Pattern**: Producer-consumer for log ingestion

2. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Long-term log storage, archival
   - **Why**: Cost-effective, durable storage
   - **Retention**: Logs older than 30 days

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Metrics extraction, time-series analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Metrics over time, aggregations

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: High-volume log storage (alternative)
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `logs_by_timestamp`, `logs_by_service`

5. **Redis** (In-Memory Cache)
   - **Use Case**: Recent logs cache, real-time dashboards
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Recent logs (1 hour)

**Database Architecture**:
```
Log Entry → Log Agent → Kafka → Elasticsearch
    ├── Elasticsearch → Log storage, search, analytics
    ├── Kafka → Ingestion pipeline, buffering
    ├── S3 → Long-term storage, archival
    ├── TimescaleDB → Metrics, time-series analytics
    ├── Cassandra → High-volume storage (alternative)
    └── Redis → Recent logs cache
```

**Selection Matrix**:
| Requirement | Elasticsearch | Kafka | S3 | TimescaleDB | Cassandra |
|------------|---------------|-------|----|-------------|-----------|
| Log Search | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| High Write Throughput | ⚠️ Moderate | ✅ Excellent | ✅ Good | ⚠️ Moderate | ✅ Excellent |
| Long-Term Storage | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |
| Real-Time Analytics | ✅ Excellent | ✅ Good | ❌ Poor | ✅ Excellent | ⚠️ Moderate |
| Cost Efficiency | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |

---

### Problem 27: Design a Metrics Collection System (Prometheus)

#### Requirements
- **Scale**: 1M+ metrics, 100K+ samples/sec, years of retention
- **Features**: Metrics collection, storage, querying, alerting
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Time-series metrics, labels, values
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Prometheus (Time-Series Database)**

**Why**:
- **Time-Series Optimized**: Efficient storage and queries
- **Label-Based**: Flexible metric labeling
- **PromQL**: Powerful query language
- **Pull Model**: Scrapes metrics from targets
- **Use Case**: Metrics storage, querying, alerting

**Secondary Databases**:

1. **TimescaleDB** (Time-Series)
   - **Use Case**: Long-term storage, Prometheus remote write
   - **Why**: PostgreSQL-based, SQL queries, long retention
   - **Schema**: Time-series tables with hypertables

2. **InfluxDB** (Time-Series)
   - **Use Case**: Alternative time-series database
   - **Why**: Optimized for time-series, high write throughput
   - **Schema**: Measurements with tags and fields

3. **Redis** (In-Memory Cache)
   - **Use Case**: Recent metrics cache, alerting state
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Recent metrics (5 min)

4. **PostgreSQL** (Relational)
   - **Use Case**: Alert rules, configuration, metadata
   - **Why**: ACID transactions, complex queries
   - **Schema**: Alert rules, configurations, metadata

5. **Grafana** (Visualization)
   - **Use Case**: Metrics visualization, dashboards
   - **Why**: Rich visualization, multiple data sources
   - **Data Source**: Connects to Prometheus, TimescaleDB, InfluxDB

**Database Architecture**:
```
Metrics → Prometheus → Storage
    ├── Prometheus → Metrics storage, querying
    ├── TimescaleDB → Long-term storage
    ├── InfluxDB → Alternative storage
    ├── Redis → Recent metrics cache
    ├── PostgreSQL → Alert rules, configuration
    └── Grafana → Visualization, dashboards
```

**Selection Matrix**:
| Requirement | Prometheus | TimescaleDB | InfluxDB | Redis | PostgreSQL |
|------------|------------|-------------|----------|-------|------------|
| Time-Series Storage | ✅ Excellent | ✅ Excellent | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Query Language | ✅ Excellent | ✅ Good | ✅ Good | ❌ Poor | ✅ Good |
| Long Retention | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ❌ Poor | ✅ Excellent |
| High Write Throughput | ✅ Good | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Alerting | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |

---

### Problem 28: Design a Real-Time Analytics System

#### Requirements
- **Scale**: 1B+ events/day, 1M+ events/sec, real-time processing
- **Features**: Stream processing, aggregation, dashboards, real-time queries
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Events, metrics, aggregations, time-series
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Kafka + Stream Processing (Kafka Streams/Flink)**

**Why**:
- **Stream Processing**: Real-time event processing
- **High Throughput**: Millions of events per second
- **Windowing**: Time-based windows for aggregations
- **Stateful Processing**: Maintain state for aggregations
- **Use Case**: Real-time event processing, aggregations

**Secondary Databases**:

1. **TimescaleDB** (Time-Series)
   - **Use Case**: Aggregated metrics storage, time-series data
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Aggregated metrics over time

2. **Redis** (In-Memory Cache)
   - **Use Case**: Real-time dashboards, recent aggregations
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Recent aggregations (1-5 min)

3. **Elasticsearch** (Search Engine)
   - **Use Case**: Event search, log analytics
   - **Why**: Full-text search, aggregations
   - **Schema**: Event documents with metadata

4. **ClickHouse** (Analytical Database)
   - **Use Case**: OLAP queries, analytics, aggregations
   - **Why**: Columnar storage, fast aggregations
   - **Schema**: Events, metrics, dimensions

5. **PostgreSQL** (Relational)
   - **Use Case**: Reference data, configuration, metadata
   - **Why**: ACID transactions, complex queries
   - **Schema**: Reference data, configurations

**Database Architecture**:
```
Events → Kafka → Stream Processor → Aggregations
    ├── Kafka → Event streaming, buffering
    ├── Stream Processor → Real-time processing, aggregations
    ├── TimescaleDB → Aggregated metrics storage
    ├── Redis → Real-time dashboards, recent data
    ├── Elasticsearch → Event search, analytics
    ├── ClickHouse → OLAP queries, analytics
    └── PostgreSQL → Reference data, configuration
```

**Selection Matrix**:
| Requirement | Kafka | TimescaleDB | Redis | Elasticsearch | ClickHouse |
|------------|-------|-------------|-------|---------------|------------|
| Stream Processing | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Real-Time Aggregations | ✅ Excellent | ✅ Good | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Time-Series Storage | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |
| OLAP Queries | ❌ Poor | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent |
| Low Latency Dashboards | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |

---

### Problem 29: Design a URL Shortener (TinyURL/bit.ly)

#### Requirements
- **Scale**: 100M+ URLs/day, 100:1 read-to-write ratio, 5 years retention
- **Features**: URL shortening, redirect, analytics, expiration
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Short URLs, long URLs, metadata, analytics
- **Consistency**: Strong consistency for redirects

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Low Latency**: Sub-millisecond redirect lookups
- **High Throughput**: Millions of redirects per second
- **TTL Support**: Automatic expiration
- **Simple Key-Value**: Perfect for URL mapping
- **Use Case**: Hot URLs cache, redirect lookups

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: All URLs storage, analytics, metadata
   - **Why**: ACID transactions, complex queries, analytics
   - **Schema**: URLs, metadata, analytics, users

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: URL storage (alternative), high scale
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `urls_by_short_code`, `urls_by_user`

3. **DynamoDB** (NoSQL - Key-Value)
   - **Use Case**: Managed URL storage alternative
   - **Why**: Managed service, auto-scaling, low latency
   - **Schema**: URLs by short_code

4. **TimescaleDB** (Time-Series)
   - **Use Case**: Click analytics, time-based analytics
   - **Why**: Time-series optimization, analytics
   - **Schema**: Clicks over time, analytics

5. **MongoDB** (Document Store)
   - **Use Case**: URL metadata, analytics data
   - **Why**: Flexible schema, nested structures
   - **Schema**: URL documents with metadata, analytics

**Database Architecture**:
```
Redirect Request → API Gateway
    ├── Redis → Hot URLs cache (cache hit: < 1ms)
    ├── PostgreSQL → All URLs, analytics (cache miss)
    ├── Cassandra → URL storage (alternative)
    ├── DynamoDB → Managed URL storage
    ├── TimescaleDB → Click analytics
    └── MongoDB → URL metadata, analytics
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | DynamoDB | TimescaleDB |
|------------|-------|------------|-----------|----------|-------------|
| Redirect Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Good | ❌ Poor |
| URL Storage | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ✅ Excellent | ❌ Poor |
| Analytics | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |
| High Read Throughput | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Cost Efficiency | ⚠️ Moderate | ✅ Good | ✅ Good | ⚠️ Moderate | ✅ Good |

---

### Problem 30: Design a Rate Limiter

#### Requirements
- **Scale**: 1B+ requests/day, 1M+ requests/sec, distributed
- **Features**: Rate limiting, sliding window, token bucket, distributed coordination
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Rate limit counters, timestamps, user/IP data
- **Consistency**: Strong consistency required

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Low Latency**: Sub-millisecond rate limit checks
- **Atomic Operations**: INCR, EXPIRE for counters
- **Data Structures**: Sorted sets for sliding window
- **TTL Support**: Automatic expiration
- **Use Case**: Rate limit counters, sliding windows

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Rate limit configuration, rules, metadata
   - **Why**: ACID transactions, complex queries
   - **Schema**: Rate limit rules, configurations, metadata

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Distributed rate limit state (alternative)
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `rate_limits_by_user`, `rate_limits_by_ip`

3. **DynamoDB** (NoSQL - Key-Value)
   - **Use Case**: Managed rate limit storage
   - **Why**: Managed service, auto-scaling, low latency
   - **Schema**: Rate limits by key (user_id, ip)

4. **ZooKeeper/etcd** (Coordination)
   - **Use Case**: Distributed coordination, leader election
   - **Why**: Distributed consensus, coordination
   - **Use Case**: Multi-region rate limiting coordination

**Database Architecture**:
```
Request → Rate Limiter → Redis
    ├── Redis → Rate limit counters, sliding windows
    ├── PostgreSQL → Rate limit rules, configuration
    ├── Cassandra → Distributed state (alternative)
    ├── DynamoDB → Managed storage
    └── ZooKeeper → Distributed coordination
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | DynamoDB | ZooKeeper |
|------------|-------|------------|-----------|----------|-----------|
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Good | ⚠️ Moderate |
| Atomic Operations | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Sliding Window | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor |
| Distributed Coordination | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |
| High Throughput | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |

---

## Database Selection Framework (Continued)

### Analytics & Monitoring Patterns

**Pattern 1: Time-Series Data**
- **Requirement**: Time-ordered data, metrics, analytics
- **Databases**: Prometheus, TimescaleDB, InfluxDB, ClickHouse
- **Use Cases**: Metrics, monitoring, analytics, IoT

**Pattern 2: Log Aggregation**
- **Requirement**: High-volume logs, search, analytics
- **Databases**: Elasticsearch, Splunk, Loki
- **Use Cases**: Application logs, system logs, audit logs

**Pattern 3: Stream Processing**
- **Requirement**: Real-time processing, aggregations
- **Databases**: Kafka, Kafka Streams, Flink, Spark Streaming
- **Use Cases**: Real-time analytics, event processing

**Pattern 4: Rate Limiting**
- **Requirement**: Low latency, atomic operations
- **Databases**: Redis, DynamoDB, in-memory solutions
- **Use Cases**: API rate limiting, DDoS protection

---

## Summary

**Problems Covered (26-30)**:
26. Distributed Logging System → Elasticsearch, Kafka, S3, TimescaleDB, Cassandra
27. Metrics Collection System → Prometheus, TimescaleDB, InfluxDB, Redis, PostgreSQL
28. Real-Time Analytics System → Kafka, TimescaleDB, Redis, Elasticsearch, ClickHouse
29. URL Shortener → Redis, PostgreSQL, Cassandra, DynamoDB, TimescaleDB
30. Rate Limiter → Redis, PostgreSQL, Cassandra, DynamoDB, ZooKeeper

**Key Patterns Identified**:
- **Time-Series**: Prometheus, TimescaleDB, InfluxDB for metrics
- **Log Aggregation**: Elasticsearch for log search and analytics
- **Stream Processing**: Kafka for real-time event processing
- **Rate Limiting**: Redis for low-latency rate limiting
- **High-Volume Writes**: Cassandra, Kafka for high write throughput

**Next**: Problems 31-35 (Real-Time & Gaming Systems)

