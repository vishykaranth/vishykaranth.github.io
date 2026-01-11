# Top 100 System Design Problems with Database Selection Criteria - Part 12

## Problems 56-60: Additional E-Commerce Systems

### Problem 56: Design a Shopping Cart System

#### Requirements
- **Scale**: 100M+ users, 10M+ active carts, < 50ms latency
- **Features**: Add/remove items, quantity updates, expiration, persistence
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Cart items, quantities, prices, metadata, timestamps
- **Consistency**: Strong consistency for cart operations

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Low Latency**: Sub-millisecond cart operations
- **High Throughput**: Millions of cart operations per second
- **TTL Support**: Automatic cart expiration
- **Data Structures**: Hashes for cart items, sorted sets for totals
- **Use Case**: Active shopping carts, cart operations, session storage

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Persistent cart storage, cart history, checkout data
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Carts, cart items, checkout, orders

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Cart storage (alternative), cart history
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `carts_by_user`, `cart_items_by_cart`

3. **DynamoDB** (NoSQL - Key-Value)
   - **Use Case**: Managed cart storage alternative
   - **Why**: Managed service, auto-scaling, low latency
   - **Schema**: Carts by user_id, cart items

4. **MongoDB** (Document Store)
   - **Use Case**: Cart documents, nested cart items
   - **Why**: Flexible schema, nested structures
   - **Schema**: Cart documents with items, metadata

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Cart analytics, abandonment metrics
   - **Why**: Time-series optimization, analytics
   - **Schema**: Cart events over time, abandonment patterns

**Database Architecture**:
```
Cart Operation → Cart Service
    ├── Redis → Active shopping carts, cart operations
    ├── PostgreSQL → Persistent cart storage, cart history
    ├── Cassandra → Cart storage (alternative), cart history
    ├── DynamoDB → Managed cart storage
    ├── MongoDB → Cart documents, nested items
    └── TimescaleDB → Cart analytics, abandonment metrics
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | DynamoDB | MongoDB |
|------------|-------|------------|-----------|----------|---------|
| Cart Operations | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Good | ⚠️ Moderate |
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Good | ⚠️ Moderate |
| Cart Persistence | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| Cart History | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |
| Cart Analytics | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |

---

### Problem 57: Design an Inventory Management System

#### Requirements
- **Scale**: 10M+ products, 1M+ inventory updates/day, real-time stock
- **Features**: Stock tracking, reservations, low-stock alerts, multi-warehouse
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Products, inventory, reservations, warehouses, transactions
- **Consistency**: Strong consistency required (inventory)

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for inventory updates, reservations
- **Strong Consistency**: Required for stock accuracy
- **Complex Queries**: Multi-warehouse queries, reservations, reporting
- **Relationships**: Products, warehouses, inventory, reservations
- **Use Case**: Inventory, products, warehouses, reservations, transactions

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Stock cache, frequently accessed products, low-stock alerts
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Stock cache (1 min), product cache (5 min)

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Inventory history, transaction logs, audit trail
   - **Why**: High write throughput, time-series data
   - **Schema**: `inventory_by_product`, `transactions_by_timestamp`

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Inventory metrics, stock trends, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Inventory changes over time, stock trends

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Product search, inventory search, analytics
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Product documents, inventory documents

5. **MongoDB** (Document Store)
   - **Use Case**: Product details, nested inventory data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Product documents with inventory, variants

**Database Architecture**:
```
Inventory Update → Inventory Service
    ├── PostgreSQL → Inventory, products, warehouses (ACID)
    ├── Redis → Stock cache, product cache, alerts
    ├── Cassandra → Inventory history, transaction logs
    ├── TimescaleDB → Inventory metrics, stock trends
    ├── Elasticsearch → Product search, inventory search
    └── MongoDB → Product details, nested data
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | Cassandra | TimescaleDB | Elasticsearch |
|------------|------------|-------|-----------|-------------|---------------|
| Inventory Management | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Stock Accuracy | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Inventory History | ✅ Excellent | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Product Search | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |
| Inventory Analytics | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |

---

### Problem 58: Design a Recommendation Engine

#### Requirements
- **Scale**: 1B+ users, 100M+ items, real-time recommendations
- **Features**: Collaborative filtering, content-based, hybrid, personalization
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: User preferences, item features, interactions, embeddings
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Recommendation Cache**: Pre-computed recommendations for users
- **Low Latency**: Sub-millisecond recommendation retrieval
- **High Throughput**: Millions of recommendation requests per second
- **Data Structures**: Sorted sets for ranked recommendations
- **Use Case**: Cached recommendations, hot recommendations, user preferences

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: User data, item metadata, interaction history
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, items, interactions, preferences

2. **ML Feature Store** (Specialized)
   - **Use Case**: ML features, user embeddings, item embeddings
   - **Why**: Feature serving, model predictions, vector storage
   - **Tools**: Redis, Cassandra, or specialized feature stores (Pinecone, Weaviate)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: User interactions, item interactions, interaction history
   - **Why**: High write throughput, time-series data
   - **Schema**: `interactions_by_user`, `interactions_by_item`

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Item search, content-based recommendations
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Item documents with features, metadata

5. **Vector Database** (Specialized)
   - **Use Case**: User embeddings, item embeddings, similarity search
   - **Why**: Efficient vector similarity search, nearest neighbor
   - **Tools**: Pinecone, Weaviate, Milvus, Qdrant

**Database Architecture**:
```
Recommendation Request → Recommendation Service
    ├── Redis → Cached recommendations, user preferences
    ├── PostgreSQL → User data, item metadata, interactions
    ├── ML Feature Store → ML features, embeddings
    ├── Cassandra → User interactions, interaction history
    ├── Elasticsearch → Item search, content-based
    └── Vector Database → User/item embeddings, similarity search
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | ML Feature Store | Cassandra | Vector DB |
|------------|-------|------------|-------------------|-----------|-----------|
| Recommendation Cache | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| User Data | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor |
| ML Features | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Interaction History | ❌ Poor | ✅ Excellent | ❌ Poor | ✅ Excellent | ❌ Poor |
| Similarity Search | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 59: Design a Review System

#### Requirements
- **Scale**: 100M+ products, 1B+ reviews, 10M+ reviews/day
- **Features**: Review submission, ratings, moderation, helpfulness, search
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Reviews, ratings, metadata, moderation status, helpfulness
- **Consistency**: Strong consistency for moderation

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for review submission, moderation
- **Complex Queries**: Review filtering, sorting, aggregations
- **Relationships**: Products, reviews, users, ratings
- **Full-Text Search**: Review content search
- **Use Case**: Reviews, ratings, moderation, metadata

**Secondary Databases**:

1. **Elasticsearch** (Search Engine)
   - **Use Case**: Review search, content search, relevance ranking
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Review documents with content, ratings, metadata

2. **Redis** (In-Memory Cache)
   - **Use Case**: Popular reviews cache, recent reviews, rating cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Popular reviews (1 hour), recent reviews (30 min)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Review storage (alternative), review history
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `reviews_by_product`, `reviews_by_user`

4. **MongoDB** (Document Store)
   - **Use Case**: Review documents, nested review data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Review documents with content, ratings, metadata

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Review metrics, rating trends, analytics
   - **Why**: Time-series optimization, analytics
   - **Schema**: Review submissions over time, rating trends

**Database Architecture**:
```
Review Submission → Review Service
    ├── PostgreSQL → Reviews, ratings, moderation (ACID)
    ├── Elasticsearch → Review search, content search
    ├── Redis → Popular reviews cache, rating cache
    ├── Cassandra → Review storage (alternative), review history
    ├── MongoDB → Review documents, nested data
    └── TimescaleDB → Review metrics, rating trends
```

**Selection Matrix**:
| Requirement | PostgreSQL | Elasticsearch | Redis | Cassandra | MongoDB |
|------------|------------|---------------|-------|-----------|---------|
| Review Storage | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ✅ Excellent |
| Review Search | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Rating Aggregations | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Review Moderation | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Review Analytics | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |

---

### Problem 60: Design a Payment Gateway

#### Requirements
- **Scale**: 1M+ transactions/day, $1B+ processed/month, < 500ms latency
- **Features**: Payment processing, multiple payment methods, fraud detection, webhooks
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Transactions, payments, customers, cards, fraud data
- **Consistency**: Strong consistency required (financial data)

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for financial transactions
- **Strong Consistency**: Required for payments, balances
- **Complex Queries**: Reconciliation, reporting, analytics
- **Audit Trail**: Complete transaction history
- **Use Case**: Transactions, payments, customers, accounts

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Payment token cache, fraud check cache, rate limiting
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Payment tokens (5 min), fraud cache (1 hour)

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Transaction logs, audit trail, analytics
   - **Why**: High write throughput, time-series data
   - **Schema**: `transactions_by_customer`, `transactions_by_timestamp`

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Transaction metrics, fraud patterns, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Transaction metrics over time, fraud patterns

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Transaction search, fraud pattern search
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Transaction documents, fraud events

5. **Message Queue (Kafka/RabbitMQ)**
   - **Use Case**: Payment events, webhooks, async processing
   - **Why**: Reliable delivery, event streaming
   - **Pattern**: Event-driven architecture for payments

**Database Architecture**:
```
Payment Request → Payment Gateway
    ├── PostgreSQL → Transactions, payments, customers (ACID)
    ├── Redis → Payment tokens, fraud cache, rate limiting
    ├── Cassandra → Transaction logs, audit trail
    ├── TimescaleDB → Transaction metrics, fraud patterns
    ├── Elasticsearch → Transaction search, fraud search
    └── Message Queue → Payment events, webhooks
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | Cassandra | TimescaleDB | Elasticsearch |
|------------|------------|-------|-----------|-------------|---------------|
| ACID Transactions | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Payment Processing | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Fraud Detection | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |
| Transaction Search | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |
| Audit Trail | ✅ Excellent | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |

---

## Database Selection Framework (Continued)

### Additional E-Commerce Patterns

**Pattern 1: Shopping Cart**
- **Requirement**: Low latency, high throughput, expiration
- **Databases**: Redis (active carts), PostgreSQL (persistence)
- **Use Cases**: E-commerce carts, session storage

**Pattern 2: Inventory Management**
- **Requirement**: ACID, strong consistency, real-time stock
- **Databases**: PostgreSQL (inventory), Redis (cache)
- **Use Cases**: Stock tracking, reservations, multi-warehouse

**Pattern 3: Recommendations**
- **Requirement**: ML features, embeddings, similarity search
- **Databases**: Redis (cache), Vector DB (embeddings), ML stores
- **Use Cases**: Product recommendations, personalized content

**Pattern 4: Reviews & Ratings**
- **Requirement**: Search, aggregations, moderation
- **Databases**: PostgreSQL (storage), Elasticsearch (search)
- **Use Cases**: Product reviews, ratings, moderation

---

## Summary

**Problems Covered (56-60)**:
56. Shopping Cart System → Redis, PostgreSQL, Cassandra, DynamoDB, MongoDB
57. Inventory Management → PostgreSQL, Redis, Cassandra, TimescaleDB, Elasticsearch
58. Recommendation Engine → Redis, PostgreSQL, ML Feature Store, Cassandra, Vector DB
59. Review System → PostgreSQL, Elasticsearch, Redis, Cassandra, MongoDB
60. Payment Gateway → PostgreSQL, Redis, Cassandra, TimescaleDB, Elasticsearch

**Key Patterns Identified**:
- **Shopping Cart**: Redis for active carts, PostgreSQL for persistence
- **Inventory**: PostgreSQL for ACID guarantees, Redis for cache
- **Recommendations**: Redis for cache, Vector DB for embeddings
- **Reviews**: PostgreSQL for storage, Elasticsearch for search
- **Payment Gateway**: PostgreSQL for transactions, Redis for tokens

**Next**: Problems 61-65 (Additional Content Systems)

