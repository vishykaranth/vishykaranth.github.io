# Top 100 System Design Problems with Database Selection Criteria - Part 4

## Problems 16-20: Search & Discovery Systems

### Problem 16: Design a Web Search Engine (Google)

#### Requirements
- **Scale**: 5B+ users, 100B+ web pages, 3.5B+ searches/day
- **Features**: Web crawling, indexing, ranking, query processing
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Web pages, indexes, rankings, query logs
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Distributed Search Engine (Elasticsearch/Solr)**

**Why**:
- **Inverted Index**: Efficient full-text search
- **Distributed**: Horizontal scaling across nodes
- **Relevance Ranking**: TF-IDF, BM25, ML-based ranking
- **Schema**: Document-based with fields, analyzers
- **Use Case**: Search index, document storage, ranking

**Secondary Databases**:

1. **BigTable/HBase** (NoSQL - Wide Column)
   - **Use Case**: Web page storage, crawl data
   - **Why**: Massive scale, column-family storage
   - **Schema**: Pages by URL, content, metadata

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Query logs, click data, analytics
   - **Why**: High write throughput, time-series data
   - **Schema**: `queries_by_timestamp`, `clicks_by_query`

3. **PostgreSQL** (Relational)
   - **Use Case**: Crawl queue, URL management, metadata
   - **Why**: ACID transactions, complex queries
   - **Schema**: URLs, crawl status, metadata

4. **Redis** (In-Memory Cache)
   - **Use Case**: Popular queries cache, autocomplete
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Query cache (1 hour), autocomplete (5 min)

5. **HDFS/Distributed File System**
   - **Use Case**: Raw web page storage, backup
   - **Why**: Cost-effective for large-scale storage
   - **Size**: Petabytes of web data

6. **Graph Database (Neo4j)**
   - **Use Case**: PageRank calculation, link graph
   - **Why**: Efficient graph algorithms, link analysis
   - **Schema**: Web pages as nodes, links as relationships

**Database Architecture**:
```
Search Query → API Gateway
    ├── Elasticsearch → Search index, ranking
    ├── BigTable → Web page storage
    ├── Cassandra → Query logs, analytics
    ├── PostgreSQL → Crawl queue, metadata
    ├── Redis → Query cache, autocomplete
    ├── HDFS → Raw page storage
    └── Neo4j → Link graph, PageRank
```

**Selection Matrix**:
| Requirement | Elasticsearch | BigTable | Cassandra | PostgreSQL | Redis |
|------------|---------------|----------|-----------|------------|-------|
| Full-Text Search | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Web Page Storage | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Query Logs | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |
| Link Graph | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Query Cache | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent |

---

### Problem 17: Design a Distributed Search System (Elasticsearch)

#### Requirements
- **Scale**: 1B+ documents, 100K+ queries/sec, petabyte-scale
- **Features**: Distributed indexing, query routing, sharding, replication
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Documents, indexes, queries, aggregations
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Elasticsearch (Distributed Search)**

**Why**:
- **Distributed Architecture**: Sharding and replication
- **Inverted Index**: Efficient full-text search
- **Real-Time**: Near real-time indexing
- **Aggregations**: Complex analytics and aggregations
- **Use Case**: Document search, analytics, log analysis

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Metadata, configuration, user data
   - **Why**: ACID transactions, complex queries
   - **Schema**: Users, indexes, configurations

2. **Redis** (In-Memory Cache)
   - **Use Case**: Query cache, popular queries, aggregations cache
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Query cache (5 min), aggregations (1 hour)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Document source data, time-series data
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `documents_by_source`, `documents_by_timestamp`

4. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Snapshot storage, backup
   - **Why**: Cost-effective, durable storage
   - **Size**: Snapshots can be large

5. **Kafka** (Message Queue)
   - **Use Case**: Document ingestion pipeline
   - **Why**: High throughput, reliable delivery
   - **Pattern**: Producer-consumer for indexing

**Database Architecture**:
```
Document → Kafka → Elasticsearch Cluster
    ├── Elasticsearch → Search index (sharded, replicated)
    ├── PostgreSQL → Metadata, configuration
    ├── Redis → Query cache, aggregations
    ├── Cassandra → Source data
    ├── S3 → Snapshots, backup
    └── Kafka → Ingestion pipeline
```

**Selection Matrix**:
| Requirement | Elasticsearch | PostgreSQL | Redis | Cassandra | S3 |
|------------|---------------|------------|-------|-----------|----|
| Full-Text Search | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Distributed Indexing | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Query Routing | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Aggregations | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor |
| Snapshot Storage | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 18: Design a Typeahead/Autocomplete System

#### Requirements
- **Scale**: 1B+ users, 10B+ queries/day, < 10ms latency
- **Features**: Prefix matching, ranking, real-time updates, personalization
- **Read/Write Ratio**: 10000:1 (reads dominate)
- **Data Types**: Query strings, frequencies, user history
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Low Latency**: Sub-millisecond response time required
- **Trie Structure**: Redis sorted sets for prefix matching
- **High Throughput**: Millions of queries per second
- **Real-Time**: Fast updates for trending queries
- **Use Case**: Autocomplete suggestions, query cache

**Secondary Databases**:

1. **Elasticsearch** (Search Engine)
   - **Use Case**: Query ranking, relevance scoring
   - **Why**: Full-text search, relevance ranking
   - **Schema**: Query documents with frequencies, metadata

2. **PostgreSQL** (Relational)
   - **Use Case**: Query history, user preferences, analytics
   - **Why**: ACID transactions, complex queries
   - **Schema**: Queries, frequencies, user history

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Query logs, frequency updates
   - **Why**: High write throughput, time-series data
   - **Schema**: `queries_by_prefix`, `frequencies_by_query`

4. **Trie Data Structure** (Custom)
   - **Use Case**: Prefix matching, efficient lookups
   - **Why**: O(m) lookup time where m is prefix length
   - **Implementation**: In-memory trie, Redis sorted sets

5. **MongoDB** (Document Store)
   - **Use Case**: Query metadata, user personalization
   - **Why**: Flexible schema, nested structures
   - **Schema**: Query documents with metadata, user preferences

**Database Architecture**:
```
Query Prefix → API Gateway
    ├── Redis → Autocomplete suggestions (trie/sorted sets)
    ├── Elasticsearch → Query ranking, relevance
    ├── PostgreSQL → Query history, analytics
    ├── Cassandra → Query logs, frequencies
    └── MongoDB → Query metadata, personalization
```

**Selection Matrix**:
| Requirement | Redis | Elasticsearch | PostgreSQL | Cassandra | MongoDB |
|------------|-------|---------------|------------|-----------|---------|
| Prefix Matching | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Low Latency | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Query Ranking | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Frequency Updates | ✅ Good | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Personalization | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |

---

### Problem 19: Design a Product Search for E-Commerce

#### Requirements
- **Scale**: 100M+ products, 1M+ searches/day, < 100ms latency
- **Features**: Faceted search, filtering, sorting, relevance ranking
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Products, categories, attributes, prices, inventory
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Elasticsearch (Search Engine)**

**Why**:
- **Faceted Search**: Built-in faceting for filters
- **Relevance Ranking**: BM25, custom scoring
- **Aggregations**: Price ranges, categories, brands
- **Full-Text Search**: Product descriptions, names
- **Use Case**: Product search, filtering, sorting

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Product catalog, inventory, relationships
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Products, categories, inventory, prices

2. **Redis** (In-Memory Cache)
   - **Use Case**: Popular products cache, search results cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Popular products (1 hour), search results (5 min)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Product catalog (read-heavy), inventory updates
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `products_by_category`, `products_by_brand`

4. **MongoDB** (Document Store)
   - **Use Case**: Product details, nested attributes, variants
   - **Why**: Flexible schema, nested structures
   - **Schema**: Product documents with variants, attributes

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Price history, inventory changes, analytics
   - **Why**: Time-series optimization, analytics
   - **Schema**: Price changes, inventory history

**Database Architecture**:
```
Search Query → API Gateway
    ├── Elasticsearch → Product search, faceting, ranking
    ├── PostgreSQL → Product catalog, inventory
    ├── Redis → Popular products cache, search cache
    ├── Cassandra → Product catalog (alternative)
    ├── MongoDB → Product details, variants
    └── TimescaleDB → Price history, analytics
```

**Selection Matrix**:
| Requirement | Elasticsearch | PostgreSQL | Redis | Cassandra | MongoDB |
|------------|---------------|------------|-------|-----------|---------|
| Faceted Search | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Relevance Ranking | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Product Catalog | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |
| Inventory Management | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |
| Price History | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |

---

### Problem 20: Design a Location-Based Search (Yelp)

#### Requirements
- **Scale**: 100M+ users, 200M+ businesses, 1M+ searches/day
- **Features**: Geospatial search, ranking, reviews, recommendations
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Businesses, locations, reviews, ratings, photos
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: PostgreSQL with PostGIS (Relational)**

**Why**:
- **Geospatial Queries**: PostGIS extension for location queries
- **Distance Calculations**: Efficient distance and radius queries
- **ACID Transactions**: For reviews, ratings, bookings
- **Complex Queries**: Joins, aggregations, filtering
- **Use Case**: Business data, locations, reviews, relationships

**Secondary Databases**:

1. **Elasticsearch** (Search Engine)
   - **Use Case**: Business search, full-text search, aggregations
   - **Why**: Full-text search, geospatial search, relevance ranking
   - **Schema**: Business documents with location, reviews, ratings

2. **Redis** (In-Memory Cache)
   - **Use Case**: Popular businesses cache, nearby businesses cache
   - **Why**: Sub-millisecond latency, geospatial data structures
   - **TTL**: Popular businesses (1 hour), nearby cache (5 min)

3. **MongoDB** (Document Store)
   - **Use Case**: Business details, nested reviews, photos
   - **Why**: Flexible schema, nested structures, geospatial indexes
   - **Schema**: Business documents with reviews, photos, metadata

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Review history, check-in data, analytics
   - **Why**: High write throughput, time-series data
   - **Schema**: `reviews_by_business`, `checkins_by_user`

5. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Business photos, user photos
   - **Why**: Cost-effective, scalable, CDN integration
   - **Size**: Photos typically KBs to MBs

**Database Architecture**:
```
Search Query → API Gateway
    ├── PostgreSQL (PostGIS) → Business data, locations, reviews
    ├── Elasticsearch → Business search, geospatial search
    ├── Redis → Popular businesses cache, nearby cache
    ├── MongoDB → Business details, reviews, photos
    ├── Cassandra → Review history, check-ins
    └── S3 → Business photos, user photos
```

**Selection Matrix**:
| Requirement | PostgreSQL | Elasticsearch | Redis | MongoDB | S3 |
|------------|------------|---------------|-------|---------|----|
| Geospatial Queries | ✅ Excellent | ✅ Excellent | ✅ Good | ✅ Good | ❌ Poor |
| Business Search | ✅ Good | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Review Management | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ❌ Poor |
| Photo Storage | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate | ✅ Excellent |
| Review History | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ❌ Poor |

---

## Database Selection Framework (Continued)

### Search & Discovery Patterns

**Pattern 1: Full-Text Search**
- **Requirement**: Text search, relevance ranking
- **Databases**: Elasticsearch, Solr, PostgreSQL (full-text)
- **Use Cases**: Web search, product search, content search

**Pattern 2: Geospatial Search**
- **Requirement**: Location-based queries, distance calculations
- **Databases**: PostgreSQL (PostGIS), Elasticsearch, MongoDB, Redis
- **Use Cases**: Location search, nearby search, mapping

**Pattern 3: Autocomplete**
- **Requirement**: Prefix matching, low latency
- **Databases**: Redis (trie), Elasticsearch, custom trie
- **Use Cases**: Search autocomplete, typeahead

**Pattern 4: Faceted Search**
- **Requirement**: Multiple filters, aggregations
- **Databases**: Elasticsearch, Solr
- **Use Cases**: E-commerce filtering, product search

---

## Summary

**Problems Covered (16-20)**:
16. Web Search Engine → Elasticsearch, BigTable, Cassandra, PostgreSQL, Redis
17. Distributed Search System → Elasticsearch, PostgreSQL, Redis, Cassandra, S3
18. Typeahead/Autocomplete → Redis, Elasticsearch, PostgreSQL, Cassandra, MongoDB
19. Product Search → Elasticsearch, PostgreSQL, Redis, Cassandra, MongoDB
20. Location-Based Search → PostgreSQL (PostGIS), Elasticsearch, Redis, MongoDB, S3

**Key Patterns Identified**:
- **Full-Text Search**: Elasticsearch for search capabilities
- **Geospatial**: PostgreSQL (PostGIS) or Elasticsearch for location queries
- **Autocomplete**: Redis for low-latency prefix matching
- **Faceted Search**: Elasticsearch for filtering and aggregations
- **High-Volume Logs**: Cassandra for query logs and analytics

**Next**: Problems 21-25 (Storage & File Systems)

