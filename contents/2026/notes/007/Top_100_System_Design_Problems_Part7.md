# Top 100 System Design Problems with Database Selection Criteria - Part 7

## Problems 31-35: Real-Time & Gaming Systems

### Problem 31: Design a Real-Time Multiplayer Game Server

#### Requirements
- **Scale**: 10M+ concurrent players, 1M+ game sessions, < 50ms latency
- **Features**: Game state synchronization, matchmaking, anti-cheat, leaderboards
- **Read/Write Ratio**: 1:1 (real-time updates)
- **Data Types**: Game state, player data, sessions, matchmaking queues
- **Consistency**: Strong consistency for game state, eventual for leaderboards

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Real-Time State**: Active game sessions, player positions
- **Low Latency**: Sub-millisecond access for game state
- **Pub/Sub**: Real-time event distribution to players
- **Data Structures**: Sorted sets for leaderboards, sets for matchmaking
- **Use Case**: Active game state, matchmaking queues, leaderboards

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Player accounts, game history, persistent data
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Players, games, history, achievements

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Game history, match data, analytics
   - **Why**: High write throughput, time-series data
   - **Schema**: `games_by_player`, `matches_by_timestamp`

3. **MongoDB** (Document Store)
   - **Use Case**: Game configurations, player profiles, nested data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Game configs, player profiles, game metadata

4. **TimescaleDB** (Time-Series)
   - **Use Case**: Game metrics, performance data, analytics
   - **Why**: Time-series optimization, analytics
   - **Schema**: Game metrics over time, performance data

5. **Neo4j** (Graph Database)
   - **Use Case**: Player relationships, social graph, team matching
   - **Why**: Efficient graph queries, relationship matching
   - **Schema**: Player nodes, friend/team relationships

**Database Architecture**:
```
Game Action → Game Server → Redis
    ├── Redis → Active game state, matchmaking, leaderboards
    ├── PostgreSQL → Player accounts, game history
    ├── Cassandra → Game history, match data
    ├── MongoDB → Game configs, player profiles
    ├── TimescaleDB → Game metrics, analytics
    └── Neo4j → Player relationships, social graph
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | MongoDB | TimescaleDB |
|------------|-------|------------|-----------|---------|-------------|
| Real-Time State | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Game History | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |
| Leaderboards | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Player Relationships | ❌ Poor | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ❌ Poor |

---

### Problem 32: Design a Live Streaming Platform (Twitch)

#### Requirements
- **Scale**: 100M+ users, 10M+ concurrent viewers, 1M+ streams/day
- **Features**: Video streaming, chat, donations, moderation, VOD
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Streams, chat messages, users, donations, VOD
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Live Chat**: Real-time chat messages, pub/sub
- **Active Streams**: Active stream metadata, viewer counts
- **Low Latency**: Sub-millisecond chat delivery
- **Data Structures**: Lists for chat, sorted sets for viewer counts
- **Use Case**: Live chat, active streams, real-time metrics

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: User accounts, stream metadata, subscriptions
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, streams, subscriptions, donations

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Chat history, stream history, analytics
   - **Why**: High write throughput, time-series data
   - **Schema**: `chat_by_stream`, `streams_by_user`, `viewers_by_stream`

3. **S3/Blob Storage** (Object Storage)
   - **Use Case**: VOD storage, stream recordings
   - **Why**: Cost-effective, scalable, CDN integration
   - **Size**: VOD files can be GBs

4. **MongoDB** (Document Store)
   - **Use Case**: Stream metadata, chat moderation, nested data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Stream documents with metadata, chat logs

5. **Elasticsearch** (Search Engine)
   - **Use Case**: Stream search, user search, content discovery
   - **Why**: Full-text search, relevance ranking
   - **Schema**: Stream documents, user documents

**Database Architecture**:
```
Stream Event → API Gateway
    ├── Redis → Live chat, active streams, real-time metrics
    ├── PostgreSQL → Users, stream metadata, subscriptions
    ├── Cassandra → Chat history, stream history
    ├── S3 → VOD storage, recordings
    ├── MongoDB → Stream metadata, moderation
    └── Elasticsearch → Stream search, discovery
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | S3 | Elasticsearch |
|------------|-------|------------|-----------|----|---------------|
| Live Chat | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Stream Metadata | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ❌ Poor | ✅ Excellent |
| VOD Storage | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Chat History | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Stream Search | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 33: Design a Collaborative Editor (Google Docs)

#### Requirements
- **Scale**: 1B+ users, 100M+ documents, real-time collaboration
- **Features**: Real-time editing, operational transformation, versioning, comments
- **Read/Write Ratio**: 1:1 (real-time updates)
- **Data Types**: Documents, operations, versions, comments, permissions
- **Consistency**: Eventual consistency with conflict resolution

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for document operations
- **Complex Queries**: Document relationships, permissions, versions
- **Full-Text Search**: Document content search
- **Schema**: Normalized relational schema
- **Use Case**: Documents, versions, permissions, metadata

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Active document state, operational transform state
   - **Why**: Sub-millisecond latency, pub/sub for real-time updates
   - **TTL**: Active documents (session duration)

2. **Operational Transform Store** (Custom/Redis)
   - **Use Case**: Pending operations, operation queue
   - **Why**: Fast operation processing, conflict resolution
   - **Schema**: Operations queue, transformation state

3. **MongoDB** (Document Store)
   - **Use Case**: Document content, nested structures, versions
   - **Why**: Flexible schema, nested documents, versioning
   - **Schema**: Document documents with content, versions

4. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Document attachments, images, large content
   - **Why**: Cost-effective, scalable
   - **Size**: Attachments can be MBs

5. **Elasticsearch** (Search Engine)
   - **Use Case**: Document search, content search
   - **Why**: Full-text search, relevance ranking
   - **Schema**: Document content indexed for search

**Database Architecture**:
```
Edit Operation → API Gateway
    ├── PostgreSQL → Documents, versions, permissions
    ├── Redis → Active document state, operation queue
    ├── Operational Transform → Conflict resolution
    ├── MongoDB → Document content, versions
    ├── S3 → Attachments, images
    └── Elasticsearch → Document search
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | MongoDB | S3 | Elasticsearch |
|------------|------------|-------|---------|----|---------------|
| Document Storage | ✅ Excellent | ❌ Poor | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Real-Time Updates | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Operational Transform | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Version Control | ✅ Excellent | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor |
| Document Search | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent |

---

### Problem 34: Design a Real-Time Dashboard

#### Requirements
- **Scale**: 1M+ users, 1B+ data points/day, < 100ms latency
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
   - **Schema**: Dashboards, users, configurations

3. **InfluxDB** (Time-Series)
   - **Use Case**: High-frequency metrics, IoT data
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

### Problem 35: Design a Leaderboard System

#### Requirements
- **Scale**: 100M+ users, 1M+ updates/sec, < 10ms latency
- **Features**: Ranking, real-time updates, time-based leaderboards, pagination
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Scores, rankings, timestamps, user data
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Sorted Sets**: Perfect data structure for leaderboards
- **Low Latency**: Sub-millisecond ranking queries
- **Atomic Operations**: ZADD, ZRANK, ZREVRANGE for rankings
- **High Throughput**: Millions of operations per second
- **Use Case**: Global leaderboards, time-based leaderboards

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: User data, score history, persistent storage
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, scores, rankings, history

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Score history, time-based leaderboards
   - **Why**: High write throughput, time-series data
   - **Schema**: `scores_by_user`, `scores_by_timestamp`

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Historical rankings, time-based analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Rankings over time, score history

4. **MongoDB** (Document Store)
   - **Use Case**: User profiles, score metadata
   - **Why**: Flexible schema, nested structures
   - **Schema**: User documents with scores, achievements

**Database Architecture**:
```
Score Update → API Gateway
    ├── Redis → Leaderboards (sorted sets), real-time rankings
    ├── PostgreSQL → User data, score history
    ├── Cassandra → Score history, time-based data
    ├── TimescaleDB → Historical rankings, analytics
    └── MongoDB → User profiles, metadata
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | TimescaleDB | MongoDB |
|------------|-------|------------|-----------|-------------|---------|
| Ranking Queries | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Score History | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Time-Based Rankings | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| High Write Throughput | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Good |

---

## Database Selection Framework (Continued)

### Real-Time & Gaming Patterns

**Pattern 1: Real-Time State**
- **Requirement**: Sub-millisecond latency, real-time updates
- **Databases**: Redis, Memcached, in-memory solutions
- **Use Cases**: Game state, live chat, real-time dashboards

**Pattern 2: Game State Management**
- **Requirement**: Fast reads/writes, pub/sub
- **Databases**: Redis, DynamoDB
- **Use Cases**: Multiplayer games, real-time collaboration

**Pattern 3: Leaderboards**
- **Requirement**: Ranking, sorted data, low latency
- **Databases**: Redis (sorted sets), PostgreSQL (with indexes)
- **Use Cases**: Gaming leaderboards, rankings, top-N queries

**Pattern 4: Real-Time Collaboration**
- **Requirement**: Operational transform, conflict resolution
- **Databases**: Redis (operations queue), PostgreSQL (persistence)
- **Use Cases**: Collaborative editing, real-time documents

---

## Summary

**Problems Covered (31-35)**:
31. Real-Time Multiplayer Game → Redis, PostgreSQL, Cassandra, MongoDB, TimescaleDB
32. Live Streaming Platform → Redis, PostgreSQL, Cassandra, S3, Elasticsearch
33. Collaborative Editor → PostgreSQL, Redis, MongoDB, S3, Elasticsearch
34. Real-Time Dashboard → Redis, TimescaleDB, PostgreSQL, InfluxDB, Elasticsearch
35. Leaderboard System → Redis, PostgreSQL, Cassandra, TimescaleDB, MongoDB

**Key Patterns Identified**:
- **Real-Time State**: Redis for low-latency real-time data
- **Game State**: Redis for active game sessions
- **Leaderboards**: Redis sorted sets for efficient rankings
- **Collaboration**: Redis + Operational Transform for conflict resolution
- **Historical Data**: TimescaleDB, PostgreSQL for persistent storage

**Next**: Problems 36-40 (Payment & Financial Systems)

