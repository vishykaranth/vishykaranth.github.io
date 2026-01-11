# Top 100 System Design Problems with Database Selection Criteria - Part 11

## Problems 51-55: Additional Social Systems

### Problem 51: Design a Social Network Graph System

#### Requirements
- **Scale**: 1B+ users, 100B+ relationships, complex graph queries
- **Features**: Friend relationships, follower relationships, graph traversal, recommendations
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Users, relationships, edges, nodes, metadata
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Neo4j (Graph Database)**

**Why**:
- **Graph Model**: Native graph data model
- **Graph Queries**: Cypher query language for graph operations
- **Traversal Performance**: Efficient graph traversal algorithms
- **Relationship Queries**: Complex relationship queries
- **Use Case**: Social graph, friend relationships, follower relationships

**Secondary Databases**:

1. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: User data, relationship metadata, high-scale storage
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `users_by_id`, `relationships_by_user`

2. **PostgreSQL** (Relational)
   - **Use Case**: User profiles, metadata, complex queries
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, profiles, metadata, settings

3. **Redis** (In-Memory Cache)
   - **Use Case**: Friend list cache, relationship cache, hot data
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Friend lists (1 hour), relationship cache (30 min)

4. **MongoDB** (Document Store)
   - **Use Case**: User profiles, nested social data
   - **Why**: Flexible schema, nested structures
   - **Schema**: User documents with social data, metadata

5. **Elasticsearch** (Search Engine)
   - **Use Case**: User search, profile search, content discovery
   - **Why**: Full-text search, relevance ranking
   - **Schema**: User documents, profile documents

**Database Architecture**:
```
Graph Query → Graph Service
    ├── Neo4j → Social graph, relationships, graph queries
    ├── Cassandra → User data, relationship metadata
    ├── PostgreSQL → User profiles, metadata
    ├── Redis → Friend list cache, relationship cache
    ├── MongoDB → User profiles, social data
    └── Elasticsearch → User search, profile search
```

**Selection Matrix**:
| Requirement | Neo4j | Cassandra | PostgreSQL | Redis | Elasticsearch |
|------------|-------|-----------|------------|-------|---------------|
| Graph Queries | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Relationship Traversal | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| User Data Storage | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Friend List Cache | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| User Search | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent |

---

### Problem 52: Design a Messaging Queue System

#### Requirements
- **Scale**: 1B+ messages/day, 1M+ messages/sec, thousands of queues
- **Features**: Message queuing, priority, delay, dead-letter, retry
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Messages, queues, metadata, delivery receipts
- **Consistency**: At-least-once delivery

#### Database Selection Criteria

**Primary Database: Kafka/RabbitMQ (Message Queue)**

**Why**:
- **Message Queuing**: Reliable message storage and delivery
- **Delivery Guarantees**: At-least-once, exactly-once options
- **Priority Queues**: Message priority support
- **Dead-Letter Queues**: Failed message handling
- **Use Case**: Task queues, message queuing, async processing

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Queue metadata cache, priority queues, recent messages
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
    ├── Kafka/RabbitMQ → Message queuing, delivery
    ├── Redis → Queue metadata cache, priority queues
    ├── PostgreSQL → Queue metadata, configurations
    ├── Cassandra → Message storage (alternative)
    ├── DynamoDB → Managed queue alternative
    └── MongoDB → Message documents
```

**Selection Matrix**:
| Requirement | Kafka/RabbitMQ | Redis | PostgreSQL | Cassandra | DynamoDB |
|------------|----------------|-------|------------|-----------|----------|
| Message Queuing | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Good |
| Delivery Guarantees | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| Priority Queues | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Dead-Letter Queue | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| High Throughput | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |

---

### Problem 53: Design an Activity Feed System

#### Requirements
- **Scale**: 1B+ users, 10B+ activities/day, < 100ms latency
- **Features**: Activity generation, aggregation, personalization, real-time updates
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Activities, user data, aggregations, timestamps
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Feed Cache**: Pre-computed activity feeds for users
- **Low Latency**: Sub-millisecond feed retrieval
- **High Throughput**: Millions of feed requests per second
- **Data Structures**: Sorted sets for ranked feeds, lists for timelines
- **Use Case**: Cached feeds, hot activities, trending content

**Secondary Databases**:

1. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Activity storage, user activities, feed generation source
   - **Why**: High write throughput, horizontal scaling, time-series data
   - **Schema**: `activities_by_user`, `activities_by_timestamp`, `feed_source`

2. **PostgreSQL** (Relational)
   - **Use Case**: User accounts, relationships, metadata
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, relationships, settings, metadata

3. **Neo4j** (Graph Database)
   - **Use Case**: Social graph, follower relationships
   - **Why**: Efficient graph traversal, relationship queries
   - **Schema**: User nodes, follow relationships

4. **MongoDB** (Document Store)
   - **Use Case**: Activity details, nested activity data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Activity documents with metadata, nested data

5. **Elasticsearch** (Search Engine)
   - **Use Case**: Activity search, content discovery, analytics
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Activity documents with content, metadata

**Database Architecture**:
```
Activity → Activity Service
    ├── Redis → Cached feeds, hot activities
    ├── Cassandra → Activity storage, feed source
    ├── PostgreSQL → Users, relationships
    ├── Neo4j → Social graph
    ├── MongoDB → Activity details
    └── Elasticsearch → Activity search, analytics
```

**Selection Matrix**:
| Requirement | Redis | Cassandra | PostgreSQL | Neo4j | Elasticsearch |
|------------|-------|-----------|------------|-------|---------------|
| Feed Retrieval | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Activity Storage | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Social Graph | ❌ Poor | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ❌ Poor |
| Activity Search | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent |
| High Write Throughput | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |

---

### Problem 54: Design a User Profile System

#### Requirements
- **Scale**: 1B+ users, 100M+ profile updates/day, < 50ms latency
- **Features**: Profile storage, search, updates, privacy settings, preferences
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Profiles, preferences, settings, metadata, relationships
- **Consistency**: Strong consistency for updates

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for profile updates
- **Complex Queries**: Profile search, filtering, relationships
- **Relationships**: Users, profiles, preferences, settings
- **Full-Text Search**: PostgreSQL full-text search capabilities
- **Use Case**: User profiles, preferences, settings, metadata

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Profile cache, frequently accessed profiles, session data
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Profile cache (1 hour), session data (24 hours)

2. **Elasticsearch** (Search Engine)
   - **Use Case**: Profile search, user search, content discovery
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Profile documents with all fields indexed

3. **MongoDB** (Document Store)
   - **Use Case**: Profile documents, nested profile data, flexible schema
   - **Why**: Flexible schema, nested structures, JSON support
   - **Schema**: Profile documents with nested preferences, settings

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Profile storage (alternative), high-scale storage
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `profiles_by_user`, `profiles_by_attribute`

5. **Neo4j** (Graph Database)
   - **Use Case**: User relationships, social connections
   - **Why**: Efficient graph queries, relationship traversal
   - **Schema**: User nodes, relationship edges

**Database Architecture**:
```
Profile Request → Profile Service
    ├── PostgreSQL → User profiles, preferences, settings (ACID)
    ├── Redis → Profile cache, frequently accessed profiles
    ├── Elasticsearch → Profile search, user search
    ├── MongoDB → Profile documents, nested data
    ├── Cassandra → Profile storage (alternative)
    └── Neo4j → User relationships, social graph
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | Elasticsearch | MongoDB | Cassandra |
|------------|------------|-------|---------------|---------|-----------|
| Profile Storage | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |
| Profile Search | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor |
| Profile Cache | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Profile Updates | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |
| User Relationships | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |

---

### Problem 55: Design a Content Moderation System

#### Requirements
- **Scale**: 1B+ content items/day, real-time moderation, ML-based detection
- **Features**: Automated moderation, manual review, ML models, policy enforcement
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Content, moderation decisions, policies, ML features, audit logs
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for moderation decisions
- **Complex Queries**: Policy queries, moderation history, reporting
- **Relationships**: Content, policies, decisions, users
- **Audit Trail**: Complete moderation history
- **Use Case**: Moderation decisions, policies, audit logs, metadata

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Policy cache, ML feature cache, recent decisions
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Policy cache (1 hour), ML features (5 min)

2. **Elasticsearch** (Search Engine)
   - **Use Case**: Content search, moderation search, analytics
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Content documents, moderation events

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Content storage, moderation logs, audit trail
   - **Why**: High write throughput, time-series data
   - **Schema**: `content_by_timestamp`, `moderation_logs_by_content`

4. **ML Feature Store** (Specialized)
   - **Use Case**: ML features, model predictions, embeddings
   - **Why**: Feature serving, model predictions
   - **Tools**: Redis, Cassandra, or specialized feature stores

5. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Content storage (images, videos), backups
   - **Why**: Cost-effective, scalable, CDN integration
   - **Size**: Media files can be large

**Database Architecture**:
```
Content → Moderation Service
    ├── PostgreSQL → Moderation decisions, policies, audit logs
    ├── Redis → Policy cache, ML feature cache
    ├── Elasticsearch → Content search, moderation search
    ├── Cassandra → Content storage, moderation logs
    ├── ML Feature Store → ML features, predictions
    └── S3 → Content storage, backups
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | Elasticsearch | Cassandra | ML Feature Store |
|------------|------------|-------|---------------|-----------|------------------|
| Moderation Decisions | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Policy Management | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Content Search | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor |
| ML Feature Serving | ❌ Poor | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ✅ Excellent |
| Audit Trail | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ❌ Poor |

---

## Database Selection Framework (Continued)

### Additional Social System Patterns

**Pattern 1: Social Graph**
- **Requirement**: Relationship queries, graph traversal
- **Databases**: Neo4j, ArangoDB, Amazon Neptune
- **Use Cases**: Social networks, friend recommendations, relationship queries

**Pattern 2: Activity Feeds**
- **Requirement**: High read throughput, real-time updates
- **Databases**: Redis (cache), Cassandra (storage)
- **Use Cases**: News feeds, activity streams, timelines

**Pattern 3: User Profiles**
- **Requirement**: Complex queries, search, relationships
- **Databases**: PostgreSQL, Elasticsearch, MongoDB
- **Use Cases**: User profiles, preferences, settings

**Pattern 4: Content Moderation**
- **Requirement**: ML integration, audit trail, search
- **Databases**: PostgreSQL, Elasticsearch, ML stores
- **Use Cases**: Content moderation, policy enforcement, ML-based detection

---

## Summary

**Problems Covered (51-55)**:
51. Social Network Graph → Neo4j, Cassandra, PostgreSQL, Redis, Elasticsearch
52. Messaging Queue System → Kafka/RabbitMQ, Redis, PostgreSQL, Cassandra, DynamoDB
53. Activity Feed System → Redis, Cassandra, PostgreSQL, Neo4j, Elasticsearch
54. User Profile System → PostgreSQL, Redis, Elasticsearch, MongoDB, Cassandra
55. Content Moderation System → PostgreSQL, Redis, Elasticsearch, Cassandra, ML Feature Store

**Key Patterns Identified**:
- **Social Graph**: Neo4j for graph queries and traversal
- **Activity Feeds**: Redis for caching, Cassandra for storage
- **User Profiles**: PostgreSQL for complex queries, Elasticsearch for search
- **Content Moderation**: PostgreSQL for decisions, ML stores for features
- **Message Queuing**: Kafka/RabbitMQ for reliable delivery

**Next**: Problems 56-60 (Additional E-Commerce Systems)

