# Top 100 System Design Problems with Database Selection Criteria - Part 1

## Overview

This document provides the top 100 system design problems with detailed database selection criteria based on use cases and requirements. Each problem includes database recommendations with justifications.

---

## Problems 1-5: Social & Communication Systems

### Problem 1: Design a Social Media Platform (Facebook/Twitter)

#### Requirements
- **Scale**: 1B+ users, 500M DAU
- **Features**: Posts, comments, likes, shares, news feed, timeline
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Text, images, videos, relationships
- **Consistency**: Eventual consistency acceptable for most features

#### Database Selection Criteria

**Primary Database: Graph Database (Neo4j)**

**Why**:
- **Social Graph**: Relationships (friends, followers) are core
- **Graph Queries**: "Friends of friends", "Mutual connections" are natural
- **Traversal Performance**: Efficient graph traversal
- **Relationship Queries**: Complex relationship queries

**Secondary Databases**:

1. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: User posts, timeline storage
   - **Why**: High write throughput, horizontal scaling, eventual consistency
   - **Schema**: `posts_by_user`, `timeline_by_user`

2. **Redis** (In-Memory Cache)
   - **Use Case**: News feed cache, session storage
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: 5-15 minutes for feed cache

3. **PostgreSQL** (Relational)
   - **Use Case**: User profiles, authentication, metadata
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, authentication, settings

4. **MongoDB** (Document Store)
   - **Use Case**: Comments, nested structures
   - **Why**: Flexible schema, nested documents, good for comments
   - **Schema**: Posts with embedded comments

**Database Architecture**:
```
User Actions → API Gateway
    ├── Graph DB (Neo4j) → Social relationships
    ├── Cassandra → Posts, timeline
    ├── Redis → Feed cache, sessions
    ├── PostgreSQL → User data, auth
    └── MongoDB → Comments, nested data
```

**Selection Matrix**:
| Requirement | Neo4j | Cassandra | Redis | PostgreSQL | MongoDB |
|------------|-------|-----------|-------|------------|---------|
| Social Graph | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Write Throughput | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Good |
| Read Latency | ✅ Good | ⚠️ Moderate | ✅ Excellent | ✅ Good | ✅ Good |
| Consistency | Eventual | Eventual | Strong | Strong | Eventual |
| Scalability | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |

---

### Problem 2: Design a Chat Application (WhatsApp/Slack)

#### Requirements
- **Scale**: 2B+ users, 1B+ messages/day
- **Features**: 1-on-1 chat, group chat, file sharing, presence
- **Read/Write Ratio**: 1:1 (real-time messaging)
- **Data Types**: Text, media files, presence status
- **Consistency**: Strong consistency for message delivery

#### Database Selection Criteria

**Primary Database: Cassandra (NoSQL - Wide Column)**

**Why**:
- **High Write Throughput**: Millions of messages per second
- **Horizontal Scaling**: Linear scaling with nodes
- **Time-Series Data**: Messages are time-ordered
- **Partitioning**: Partition by chat_id for efficient queries
- **Schema**: `messages_by_chat (chat_id, timestamp, message_id)`

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Active conversations, presence, recent messages
   - **Why**: Sub-millisecond latency, pub/sub for real-time
   - **TTL**: Active chats cached, presence updates in real-time

2. **PostgreSQL** (Relational)
   - **Use Case**: User accounts, chat metadata, contacts
   - **Why**: ACID transactions, relationships, complex queries
   - **Schema**: Users, chats, participants, contacts

3. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Media files (images, videos, documents)
   - **Why**: Cost-effective, scalable, CDN integration
   - **Size**: Files > 1MB

4. **MongoDB** (Document Store)
   - **Use Case**: Chat metadata, group info, settings
   - **Why**: Flexible schema, nested structures
   - **Schema**: Chat documents with participants, settings

**Database Architecture**:
```
Message → API Gateway
    ├── Cassandra → Message storage (partitioned by chat_id)
    ├── Redis → Active chats, presence, pub/sub
    ├── PostgreSQL → User data, chat metadata
    ├── S3 → Media files
    └── MongoDB → Chat settings, group info
```

**Selection Matrix**:
| Requirement | Cassandra | Redis | PostgreSQL | S3 | MongoDB |
|------------|-----------|-------|------------|----|---------|
| Message Storage | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Real-Time Updates | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Write Throughput | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Good | ✅ Good |
| Presence | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Media Storage | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |

---

### Problem 3: Design a Video Conferencing System (Zoom/Teams)

#### Requirements
- **Scale**: 100M+ users, 10M+ concurrent meetings
- **Features**: Video/audio streaming, screen sharing, recording, chat
- **Read/Write Ratio**: 1:1 (real-time streaming)
- **Data Types**: Video streams, audio, metadata, recordings
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Real-Time State**: Active meeting state, participant lists
- **Low Latency**: Sub-millisecond access for session data
- **Pub/Sub**: Real-time event distribution
- **TTL**: Automatic session expiration
- **Use Case**: Active sessions, participant presence, signaling

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: User accounts, meeting metadata, scheduling
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, meetings, participants, recordings metadata

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Meeting history, chat messages, analytics
   - **Why**: High write throughput, time-series data
   - **Schema**: `meetings_by_user`, `chat_by_meeting`

3. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Recorded videos, screen shares
   - **Why**: Cost-effective for large files, CDN integration
   - **Size**: Videos can be GBs

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Search meeting history, transcripts
   - **Why**: Full-text search, analytics, aggregations
   - **Schema**: Meeting transcripts, metadata

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Meeting metrics, quality metrics
   - **Why**: Time-series optimization, analytics
   - **Schema**: Connection quality, bandwidth usage

**Database Architecture**:
```
Meeting Request → API Gateway
    ├── Redis → Active sessions, signaling, presence
    ├── PostgreSQL → User data, meeting metadata
    ├── Cassandra → Meeting history, chat
    ├── S3 → Recorded videos
    ├── Elasticsearch → Search, transcripts
    └── TimescaleDB → Metrics, analytics
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | S3 | Elasticsearch |
|------------|-------|------------|-----------|----|----|
| Real-Time State | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Meeting Metadata | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Video Storage | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Search | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| Analytics | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ✅ Excellent |

---

### Problem 4: Design a Notification System

#### Requirements
- **Scale**: 1B+ users, 10B+ notifications/day
- **Features**: Push, email, SMS, in-app notifications
- **Read/Write Ratio**: 10:1 (more reads)
- **Data Types**: Notification content, delivery status, preferences
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
   - **Use Case**: Notification history, delivery logs
   - **Why**: High write throughput, time-series data
   - **Schema**: `notifications_by_user`, `delivery_logs`

2. **PostgreSQL** (Relational)
   - **Use Case**: User preferences, notification settings
   - **Why**: ACID transactions, complex queries
   - **Schema**: Users, preferences, templates

3. **RabbitMQ/Kafka** (Message Queue)
   - **Use Case**: Notification delivery queue
   - **Why**: Reliable delivery, multiple consumers
   - **Pattern**: Producer-consumer for delivery

4. **MongoDB** (Document Store)
   - **Use Case**: Notification templates, content
   - **Why**: Flexible schema, nested structures
   - **Schema**: Templates with variables

**Database Architecture**:
```
Notification Request → API Gateway
    ├── Redis → Queue, rate limiting, recent notifications
    ├── Cassandra → Notification history, delivery logs
    ├── PostgreSQL → User preferences, settings
    ├── RabbitMQ → Delivery queue (push, email, SMS)
    └── MongoDB → Templates, content
```

**Selection Matrix**:
| Requirement | Redis | Cassandra | PostgreSQL | RabbitMQ | MongoDB |
|------------|-------|-----------|------------|----------|---------|
| Queue Management | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Notification History | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| User Preferences | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Delivery Queue | ✅ Good | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Templates | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent |

---

### Problem 5: Design a Friend Recommendation System

#### Requirements
- **Scale**: 1B+ users, 100M+ recommendations/day
- **Features**: Friend suggestions, mutual connections, interests
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: User profiles, relationships, interests, activity
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Neo4j (Graph Database)**

**Why**:
- **Social Graph**: Natural representation of relationships
- **Graph Algorithms**: Friend-of-friend, mutual connections
- **Traversal**: Efficient graph traversal for recommendations
- **Pattern Matching**: Complex relationship patterns
- **Use Case**: Social graph, relationship queries

**Secondary Databases**:

1. **Elasticsearch** (Search Engine)
   - **Use Case**: User profile search, interest matching
   - **Why**: Full-text search, relevance scoring
   - **Schema**: User profiles, interests, activities

2. **Redis** (In-Memory Cache)
   - **Use Case**: Cached recommendations, user activity
   - **Why**: Fast access to hot data, recommendation cache
   - **TTL**: 1-24 hours for recommendations

3. **PostgreSQL** (Relational)
   - **Use Case**: User profiles, metadata, preferences
   - **Why**: ACID transactions, complex queries
   - **Schema**: Users, profiles, settings

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: User activity logs, interaction history
   - **Why**: High write throughput, time-series data
   - **Schema**: `activity_by_user`, `interactions`

5. **Machine Learning Store** (Feature Store)
   - **Use Case**: ML features, model predictions
   - **Why**: Feature serving, model predictions
   - **Tools**: Redis, Cassandra, or specialized feature stores

**Database Architecture**:
```
Recommendation Request → API Gateway
    ├── Neo4j → Social graph, relationship queries
    ├── Elasticsearch → Profile search, interest matching
    ├── Redis → Cached recommendations, activity
    ├── PostgreSQL → User profiles, metadata
    ├── Cassandra → Activity logs, interactions
    └── Feature Store → ML features, predictions
```

**Selection Matrix**:
| Requirement | Neo4j | Elasticsearch | Redis | PostgreSQL | Cassandra |
|------------|-------|---------------|-------|------------|-----------|
| Graph Queries | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Profile Search | ❌ Poor | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Recommendation Cache | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| User Profiles | ⚠️ Moderate | ✅ Good | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Activity Logs | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |

---

## Database Selection Framework

### Decision Tree for Database Selection

```
1. Data Model?
   ├── Relationships/Graph → Neo4j, ArangoDB
   ├── Documents/Nested → MongoDB, CouchDB
   ├── Key-Value → Redis, DynamoDB
   ├── Wide Column → Cassandra, HBase
   ├── Time-Series → TimescaleDB, InfluxDB
   └── Relational → PostgreSQL, MySQL

2. Consistency Requirements?
   ├── Strong Consistency → PostgreSQL, MySQL, MongoDB (with transactions)
   └── Eventual Consistency → Cassandra, DynamoDB, MongoDB (default)

3. Scale Requirements?
   ├── < 1M users → PostgreSQL, MySQL
   ├── 1M-100M users → PostgreSQL (with replicas), MongoDB
   └── > 100M users → Cassandra, DynamoDB, MongoDB (sharded)

4. Read/Write Pattern?
   ├── Read-Heavy → Redis (cache), PostgreSQL (with replicas)
   ├── Write-Heavy → Cassandra, DynamoDB
   └── Balanced → PostgreSQL, MongoDB

5. Latency Requirements?
   ├── < 1ms → Redis (in-memory)
   ├── < 10ms → PostgreSQL, MongoDB (with cache)
   └── < 100ms → Most databases

6. Query Complexity?
   ├── Simple (key-value) → Redis, DynamoDB
   ├── Moderate (filtering) → MongoDB, Cassandra
   └── Complex (joins, aggregations) → PostgreSQL, MySQL
```

---

## Summary

**Problems Covered (1-5)**:
1. Social Media Platform → Neo4j, Cassandra, Redis, PostgreSQL, MongoDB
2. Chat Application → Cassandra, Redis, PostgreSQL, S3, MongoDB
3. Video Conferencing → Redis, PostgreSQL, Cassandra, S3, Elasticsearch
4. Notification System → Redis, Cassandra, PostgreSQL, RabbitMQ, MongoDB
5. Friend Recommendation → Neo4j, Elasticsearch, Redis, PostgreSQL, Cassandra

**Key Database Selection Criteria**:
- Data model (graph, document, relational, etc.)
- Consistency requirements (strong vs eventual)
- Scale (users, data volume)
- Read/write patterns
- Latency requirements
- Query complexity

**Next**: Problems 6-10 (E-Commerce & Marketplace Systems)

