# Top 100 System Design Problems with Database Selection Criteria - Part 17

## Problems 81-85: Additional Real-Time Systems

### Problem 81: Design a Real-Time Collaboration System

#### Requirements
- **Scale**: 100M+ users, 1M+ concurrent sessions, real-time updates
- **Features**: Real-time editing, operational transform, conflict resolution, presence
- **Read/Write Ratio**: 1:1 (real-time updates)
- **Data Types**: Documents, operations, presence, sessions, metadata
- **Consistency**: Eventual consistency with conflict resolution

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Real-Time State**: Active document state, operational transform state
- **Low Latency**: Sub-millisecond operation processing
- **Pub/Sub**: Real-time event distribution to collaborators
- **Data Structures**: Lists for operations, sorted sets for presence
- **Use Case**: Active sessions, operations queue, presence, real-time state

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Document storage, versions, permissions, metadata
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Documents, versions, permissions, metadata

2. **Operational Transform Store** (Custom/Redis)
   - **Use Case**: Pending operations, operation queue, transformation state
   - **Why**: Fast operation processing, conflict resolution
   - **Schema**: Operations queue, transformation state

3. **MongoDB** (Document Store)
   - **Use Case**: Document content, nested operations, versions
   - **Why**: Flexible schema, nested structures, versioning
   - **Schema**: Document documents with content, operations, versions

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Document history, operation history, audit trail
   - **Why**: High write throughput, time-series data
   - **Schema**: `documents_by_user`, `operations_by_document`

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Collaboration metrics, presence metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Collaboration metrics over time, presence data

**Database Architecture**:
```
Operation → Collaboration Service
    ├── Redis → Active sessions, operations queue, presence
    ├── PostgreSQL → Documents, versions, permissions
    ├── Operational Transform → Conflict resolution
    ├── MongoDB → Document content, nested operations
    ├── Cassandra → Document history, operation history
    └── TimescaleDB → Collaboration metrics, analytics
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | MongoDB | Cassandra | TimescaleDB |
|------------|-------|------------|---------|-----------|-------------|
| Real-Time Updates | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Operational Transform | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Document Storage | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ❌ Poor |
| Conflict Resolution | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Collaboration Metrics | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 82: Design a Live Chat Support System

#### Requirements
- **Scale**: 10M+ users, 1M+ chats/day, real-time messaging, agent routing
- **Features**: Real-time chat, agent assignment, queue management, history
- **Read/Write Ratio**: 1:1 (real-time messaging)
- **Data Types**: Messages, chats, agents, queues, history
- **Consistency**: Strong consistency for agent assignment

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Live Chat**: Real-time chat messages, pub/sub
- **Queue Management**: Agent queue, chat queue
- **Low Latency**: Sub-millisecond message delivery
- **Data Structures**: Lists for messages, sorted sets for queues
- **Use Case**: Active chats, message queue, agent queue, real-time state

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Chat history, agent data, user data, metadata
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Chats, messages, agents, users, metadata

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Chat history, message history, high-scale storage
   - **Why**: High write throughput, time-series data
   - **Schema**: `chats_by_user`, `messages_by_chat`, `chats_by_agent`

3. **MongoDB** (Document Store)
   - **Use Case**: Chat documents, nested messages, metadata
   - **Why**: Flexible schema, nested structures
   - **Schema**: Chat documents with messages, metadata

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Chat search, message search, content search
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Chat documents, message documents

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Chat metrics, agent metrics, response time metrics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Chat metrics over time, agent performance

**Database Architecture**:
```
Message → Chat Service
    ├── Redis → Active chats, message queue, agent queue
    ├── PostgreSQL → Chat history, agent data, user data
    ├── Cassandra → Chat history, message history
    ├── MongoDB → Chat documents, nested messages
    ├── Elasticsearch → Chat search, message search
    └── TimescaleDB → Chat metrics, agent metrics
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | MongoDB | Elasticsearch |
|------------|-------|------------|-----------|---------|---------------|
| Real-Time Chat | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Agent Queue | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Chat History | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Chat Search | ❌ Poor | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent |
| Chat Metrics | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |

---

### Problem 83: Design a Real-Time Notifications System

#### Requirements
- **Scale**: 1B+ users, 10B+ notifications/day, real-time delivery
- **Features**: Push notifications, email, SMS, in-app, delivery tracking
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Notifications, delivery status, preferences, channels
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Delivery Queue**: Fast notification queuing
- **Rate Limiting**: Per-user rate limits
- **Recent Notifications**: Cache recent notifications
- **Pub/Sub**: Real-time notification delivery
- **TTL**: Automatic expiration of old notifications

**Secondary Databases**:

1. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Notification history, delivery logs, analytics
   - **Why**: High write throughput, time-series data
   - **Schema**: `notifications_by_user`, `delivery_logs`, `notifications_by_timestamp`

2. **PostgreSQL** (Relational)
   - **Use Case**: User preferences, notification settings, metadata
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, preferences, settings, metadata

3. **RabbitMQ/Kafka** (Message Queue)
   - **Use Case**: Notification delivery queue, async processing
   - **Why**: Reliable delivery, multiple consumers, retry
   - **Pattern**: Producer-consumer for delivery

4. **MongoDB** (Document Store)
   - **Use Case**: Notification templates, content, nested data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Template documents with variables, content

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Notification metrics, delivery metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Notification metrics over time, delivery metrics

**Database Architecture**:
```
Notification Request → Notification Service
    ├── Redis → Queue, rate limiting, recent notifications
    ├── Cassandra → Notification history, delivery logs
    ├── PostgreSQL → User preferences, settings
    ├── RabbitMQ/Kafka → Delivery queue (push, email, SMS)
    ├── MongoDB → Templates, content
    └── TimescaleDB → Notification metrics, analytics
```

**Selection Matrix**:
| Requirement | Redis | Cassandra | PostgreSQL | RabbitMQ/Kafka | MongoDB |
|------------|-------|-----------|------------|---------------|---------|
| Queue Management | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Notification History | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| User Preferences | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Delivery Queue | ✅ Good | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Templates | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent |

---

### Problem 84: Design a Live Analytics Dashboard

#### Requirements
- **Scale**: 1M+ users, 1B+ data points/day, < 100ms latency, real-time updates
- **Features**: Real-time data updates, WebSocket, data aggregation, visualization
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Metrics, time-series data, aggregations, visualizations
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Real-Time Updates**: Live data for dashboards
- **Low Latency**: Sub-millisecond data retrieval
- **Pub/Sub**: Real-time event distribution via WebSocket
- **Data Structures**: Sorted sets for time-series, hashes for metrics
- **Use Case**: Real-time dashboard data, recent metrics

**Secondary Databases**:

1. **TimescaleDB** (Time-Series)
   - **Use Case**: Historical data, time-series storage, aggregations
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Metrics over time, aggregations

2. **PostgreSQL** (Relational)
   - **Use Case**: Dashboard configurations, user preferences, metadata
   - **Why**: ACID transactions, complex queries
   - **Schema**: Dashboards, users, configurations, metadata

3. **InfluxDB** (Time-Series)
   - **Use Case**: High-frequency metrics, IoT data, real-time metrics
   - **Why**: Optimized for time-series, high write throughput
   - **Schema**: Metrics, tags, fields

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Log data visualization, search-based dashboards
   - **Why**: Full-text search, aggregations
   - **Schema**: Log documents, event documents

5. **ClickHouse** (Analytical Database)
   - **Use Case**: OLAP queries, complex aggregations
   - **Why**: Columnar storage, fast aggregations
   - **Schema**: Events, metrics, dimensions

**Database Architecture**:
```
Data Update → Data Pipeline → Storage
    ├── Redis → Real-time dashboard data, recent metrics
    ├── TimescaleDB → Historical data, time-series
    ├── PostgreSQL → Dashboard configs, user preferences
    ├── InfluxDB → High-frequency metrics
    ├── Elasticsearch → Log visualization
    └── ClickHouse → OLAP queries, analytics
```

**Selection Matrix**:
| Requirement | Redis | TimescaleDB | PostgreSQL | InfluxDB | Elasticsearch |
|------------|-------|-------------|------------|----------|---------------|
| Real-Time Updates | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |
| Historical Data | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Time-Series Queries | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Dashboard Configs | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor |
| Aggregations | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |

---

### Problem 85: Design a Real-Time Monitoring System

#### Requirements
- **Scale**: 100K+ metrics/sec, 1M+ alerts/day, real-time alerting
- **Features**: Metrics collection, alerting, dashboards, anomaly detection
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Metrics, alerts, dashboards, thresholds, rules
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Prometheus (Time-Series Database)**

**Why**:
- **Metrics Collection**: Efficient metrics collection and storage
- **Time-Series Optimized**: Optimized for time-series data
- **PromQL**: Powerful query language for metrics
- **Alerting**: Built-in alerting capabilities
- **Use Case**: Metrics storage, querying, alerting

**Secondary Databases**:

1. **TimescaleDB** (Time-Series)
   - **Use Case**: Long-term storage, Prometheus remote write
   - **Why**: PostgreSQL-based, SQL queries, long retention
   - **Schema**: Time-series tables with hypertables

2. **Redis** (In-Memory Cache)
   - **Use Case**: Recent metrics cache, alert state, threshold cache
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Recent metrics (5 min), alert state (1 hour)

3. **PostgreSQL** (Relational)
   - **Use Case**: Alert rules, configurations, metadata, dashboards
   - **Why**: ACID transactions, complex queries
   - **Schema**: Alert rules, configurations, metadata, dashboards

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Log search, event search, anomaly detection
   - **Why**: Full-text search, aggregations, anomaly detection
   - **Schema**: Log documents, event documents, anomalies

5. **Grafana** (Visualization)
   - **Use Case**: Metrics visualization, dashboards
   - **Why**: Rich visualization, multiple data sources
   - **Data Source**: Connects to Prometheus, TimescaleDB, InfluxDB

**Database Architecture**:
```
Metrics → Prometheus → Storage
    ├── Prometheus → Metrics storage, querying, alerting
    ├── TimescaleDB → Long-term storage
    ├── Redis → Recent metrics cache, alert state
    ├── PostgreSQL → Alert rules, configurations
    ├── Elasticsearch → Log search, anomaly detection
    └── Grafana → Visualization, dashboards
```

**Selection Matrix**:
| Requirement | Prometheus | TimescaleDB | Redis | PostgreSQL | Elasticsearch |
|------------|------------|-------------|-------|------------|---------------|
| Metrics Collection | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Time-Series Queries | ✅ Excellent | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |
| Alerting | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Long Retention | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ✅ Excellent | ⚠️ Moderate |
| Anomaly Detection | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |

---

## Database Selection Framework (Continued)

### Additional Real-Time System Patterns

**Pattern 1: Real-Time Collaboration**
- **Requirement**: Operational transform, conflict resolution, presence
- **Databases**: Redis (state), PostgreSQL (persistence), Operational Transform
- **Use Cases**: Collaborative editing, real-time documents

**Pattern 2: Live Chat**
- **Requirement**: Real-time messaging, agent routing, queue management
- **Databases**: Redis (active chats), PostgreSQL (history), Cassandra (logs)
- **Use Cases**: Customer support, live chat, agent routing

**Pattern 3: Real-Time Notifications**
- **Requirement**: Push notifications, delivery tracking, rate limiting
- **Databases**: Redis (queue), Cassandra (history), Message Queues (delivery)
- **Use Cases**: Push notifications, email, SMS, in-app notifications

**Pattern 4: Live Dashboards**
- **Requirement**: Real-time updates, WebSocket, time-series data
- **Databases**: Redis (real-time), TimescaleDB (historical), InfluxDB (metrics)
- **Use Cases**: Real-time dashboards, live analytics, monitoring

---

## Summary

**Problems Covered (81-85)**:
81. Real-Time Collaboration → Redis, PostgreSQL, MongoDB, Cassandra, TimescaleDB
82. Live Chat Support → Redis, PostgreSQL, Cassandra, MongoDB, Elasticsearch
83. Real-Time Notifications → Redis, Cassandra, PostgreSQL, RabbitMQ/Kafka, MongoDB
84. Live Analytics Dashboard → Redis, TimescaleDB, PostgreSQL, InfluxDB, Elasticsearch
85. Real-Time Monitoring → Prometheus, TimescaleDB, Redis, PostgreSQL, Elasticsearch

**Key Patterns Identified**:
- **Real-Time Collaboration**: Redis for state, Operational Transform for conflicts
- **Live Chat**: Redis for active chats, PostgreSQL for history
- **Real-Time Notifications**: Redis for queue, Cassandra for history
- **Live Dashboards**: Redis for real-time, TimescaleDB for historical
- **Real-Time Monitoring**: Prometheus for metrics, Grafana for visualization

**Next**: Problems 86-90 (Additional Infrastructure Systems)

