# Top 100 System Design Problems with Database Selection Criteria - Part 14

## Problems 66-70: Additional Search Systems

### Problem 66: Design an Enterprise Search System

#### Requirements
- **Scale**: 1B+ documents, 100M+ searches/day, multiple data sources
- **Features**: Full-text search, faceted search, relevance ranking, connectors
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Documents, metadata, indexes, search queries
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Elasticsearch (Search Engine)**

**Why**:
- **Full-Text Search**: Efficient full-text search across documents
- **Relevance Ranking**: BM25, custom scoring, ML-based ranking
- **Faceted Search**: Built-in faceting for filters
- **Connectors**: Integration with multiple data sources
- **Use Case**: Document search, enterprise search, content discovery

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Document metadata, relationships, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Documents, metadata, configurations, relationships

2. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Original document storage, attachments
   - **Why**: Cost-effective, scalable storage
   - **Size**: Documents can be large

3. **Redis** (In-Memory Cache)
   - **Use Case**: Popular searches cache, frequently accessed documents
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Search cache (5 min), documents (1 hour)

4. **MongoDB** (Document Store)
   - **Use Case**: Document storage (alternative), nested document data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Document documents with metadata, content

5. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Document storage (alternative), high-scale storage
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `documents_by_source`, `documents_by_timestamp`

**Database Architecture**:
```
Search Query → Search Service
    ├── Elasticsearch → Document search, faceting, ranking
    ├── PostgreSQL → Document metadata, configurations
    ├── S3 → Original document storage
    ├── Redis → Search cache, popular documents
    ├── MongoDB → Document storage (alternative)
    └── Cassandra → Document storage (alternative)
```

**Selection Matrix**:
| Requirement | Elasticsearch | PostgreSQL | S3 | Redis | MongoDB |
|------------|---------------|------------|----|----|---------|
| Full-Text Search | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Relevance Ranking | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Faceted Search | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Document Storage | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ❌ Poor | ✅ Excellent |
| Search Cache | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |

---

### Problem 67: Design a Semantic Search System

#### Requirements
- **Scale**: 100M+ documents, 10M+ searches/day, vector embeddings
- **Features**: Semantic search, vector similarity, embeddings, ML models
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Documents, embeddings, vectors, metadata, queries
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Vector Database (Pinecone/Weaviate/Milvus)**

**Why**:
- **Vector Storage**: Efficient storage of high-dimensional vectors
- **Similarity Search**: Fast nearest neighbor search
- **Embeddings**: Native support for embeddings
- **ML Integration**: Direct integration with ML models
- **Use Case**: Semantic search, similarity search, embeddings

**Secondary Databases**:

1. **Elasticsearch** (Search Engine)
   - **Use Case**: Hybrid search (vector + keyword), metadata search
   - **Why**: Full-text search, vector search support, hybrid search
   - **Schema**: Document embeddings, metadata, content

2. **PostgreSQL** (Relational)
   - **Use Case**: Document metadata, relationships, configurations
   - **Why**: ACID transactions, complex queries, pgvector extension
   - **Extensions**: pgvector for vector similarity search

3. **Redis** (In-Memory Cache)
   - **Use Case**: Embedding cache, frequently accessed vectors
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Embedding cache (1 hour), vectors (30 min)

4. **MongoDB** (Document Store)
   - **Use Case**: Document storage, nested metadata
   - **Why**: Flexible schema, nested structures
   - **Schema**: Document documents with embeddings, metadata

5. **ML Feature Store** (Specialized)
   - **Use Case**: ML features, model predictions, embeddings
   - **Why**: Feature serving, model predictions
   - **Tools**: Redis, Cassandra, or specialized feature stores

**Database Architecture**:
```
Search Query → Semantic Search Service
    ├── Vector Database → Document embeddings, similarity search
    ├── Elasticsearch → Hybrid search, metadata search
    ├── PostgreSQL → Document metadata, relationships (pgvector)
    ├── Redis → Embedding cache, vectors cache
    ├── MongoDB → Document storage, metadata
    └── ML Feature Store → ML features, predictions
```

**Selection Matrix**:
| Requirement | Vector DB | Elasticsearch | PostgreSQL | Redis | MongoDB |
|------------|-----------|---------------|------------|-------|---------|
| Vector Similarity | ✅ Excellent | ✅ Good | ✅ Good | ❌ Poor | ❌ Poor |
| Semantic Search | ✅ Excellent | ✅ Good | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Hybrid Search | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Embedding Storage | ✅ Excellent | ✅ Good | ✅ Good | ⚠️ Moderate | ⚠️ Moderate |
| ML Integration | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor |

---

### Problem 68: Design an Image Search System

#### Requirements
- **Scale**: 1B+ images, 10M+ searches/day, visual similarity
- **Features**: Image search, visual similarity, reverse image search, metadata search
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Images, embeddings, vectors, metadata, search queries
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Vector Database (Pinecone/Weaviate/Milvus)**

**Why**:
- **Image Embeddings**: Efficient storage of image embeddings
- **Visual Similarity**: Fast visual similarity search
- **Reverse Image Search**: Efficient reverse image search
- **ML Integration**: Direct integration with vision models
- **Use Case**: Image embeddings, visual similarity, reverse search

**Secondary Databases**:

1. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Original image storage, thumbnails
   - **Why**: Cost-effective, scalable storage
   - **Size**: Images typically KBs to MBs

2. **Elasticsearch** (Search Engine)
   - **Use Case**: Image metadata search, hybrid search
   - **Why**: Full-text search, metadata search, aggregations
   - **Schema**: Image documents with metadata, embeddings

3. **PostgreSQL** (Relational)
   - **Use Case**: Image metadata, relationships, configurations
   - **Why**: ACID transactions, complex queries, pgvector extension
   - **Extensions**: pgvector for vector similarity search

4. **Redis** (In-Memory Cache)
   - **Use Case**: Popular images cache, embedding cache, search cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Popular images (1 hour), embeddings (30 min)

5. **MongoDB** (Document Store)
   - **Use Case**: Image metadata, nested metadata
   - **Why**: Flexible schema, nested structures
   - **Schema**: Image documents with metadata, embeddings

**Database Architecture**:
```
Image Search → Image Search Service
    ├── Vector Database → Image embeddings, visual similarity
    ├── S3 → Original image storage, thumbnails
    ├── Elasticsearch → Image metadata search, hybrid search
    ├── PostgreSQL → Image metadata, relationships (pgvector)
    ├── Redis → Popular images cache, embedding cache
    └── MongoDB → Image metadata, nested data
```

**Selection Matrix**:
| Requirement | Vector DB | S3 | Elasticsearch | PostgreSQL | Redis |
|------------|-----------|----|---------------|------------|-------|
| Visual Similarity | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ✅ Good | ❌ Poor |
| Image Storage | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Metadata Search | ❌ Poor | ❌ Poor | ✅ Excellent | ✅ Excellent | ❌ Poor |
| Reverse Image Search | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Good | ❌ Poor |
| Image Cache | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 69: Design a Voice Search System

#### Requirements
- **Scale**: 100M+ users, 1M+ voice searches/day, real-time processing
- **Features**: Speech-to-text, search, natural language understanding, results
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Audio, transcripts, search queries, results, metadata
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Elasticsearch (Search Engine)**

**Why**:
- **Full-Text Search**: Search across transcripts, content
- **Relevance Ranking**: BM25, custom scoring for voice queries
- **Natural Language**: Support for natural language queries
- **Aggregations**: Voice query analytics, aggregations
- **Use Case**: Transcript search, voice query search, results

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Voice query metadata, results, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Voice queries, results, metadata, configurations

2. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Audio file storage, transcripts
   - **Why**: Cost-effective, scalable storage
   - **Size**: Audio files can be MBs

3. **Redis** (In-Memory Cache)
   - **Use Case**: Popular voice queries cache, results cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Voice queries (1 hour), results (30 min)

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Voice query history, transcript storage
   - **Why**: High write throughput, time-series data
   - **Schema**: `voice_queries_by_user`, `transcripts_by_timestamp`

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Voice query metrics, analytics, trends
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Voice query metrics over time, analytics

**Database Architecture**:
```
Voice Query → Speech-to-Text → Search Service
    ├── Elasticsearch → Transcript search, voice query search
    ├── PostgreSQL → Voice query metadata, results
    ├── S3 → Audio file storage, transcripts
    ├── Redis → Popular queries cache, results cache
    ├── Cassandra → Voice query history, transcripts
    └── TimescaleDB → Voice query metrics, analytics
```

**Selection Matrix**:
| Requirement | Elasticsearch | PostgreSQL | S3 | Redis | Cassandra |
|------------|---------------|------------|----|-------|-----------|
| Transcript Search | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Voice Query Search | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Audio Storage | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor |
| Query Cache | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Query Analytics | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ⚠️ Moderate |

---

### Problem 70: Design a Multi-Language Search System

#### Requirements
- **Scale**: 1B+ documents, 100M+ searches/day, 100+ languages
- **Features**: Multi-language support, translation, language detection, search
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Documents, translations, language metadata, search queries
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Elasticsearch (Search Engine)**

**Why**:
- **Multi-Language**: Built-in support for multiple languages
- **Language Analyzers**: Language-specific analyzers
- **Translation**: Integration with translation services
- **Full-Text Search**: Efficient multi-language search
- **Use Case**: Multi-language document search, translation search

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Document metadata, language metadata, configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Documents, languages, translations, metadata

2. **Redis** (In-Memory Cache)
   - **Use Case**: Translation cache, popular searches cache, language cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Translation cache (1 hour), searches (30 min)

3. **MongoDB** (Document Store)
   - **Use Case**: Document storage, nested translations, flexible schema
   - **Why**: Flexible schema, nested structures, multi-language support
   - **Schema**: Document documents with translations, metadata

4. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Document storage (alternative), translation storage
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `documents_by_language`, `translations_by_document`

5. **Translation Service** (External)
   - **Use Case**: Document translation, query translation
   - **Why**: Specialized translation services
   - **Tools**: Google Translate API, AWS Translate, Azure Translator

**Database Architecture**:
```
Search Query → Multi-Language Search Service
    ├── Elasticsearch → Multi-language search, language analyzers
    ├── PostgreSQL → Document metadata, language metadata
    ├── Redis → Translation cache, popular searches cache
    ├── MongoDB → Document storage, nested translations
    ├── Cassandra → Document storage (alternative), translations
    └── Translation Service → Document translation, query translation
```

**Selection Matrix**:
| Requirement | Elasticsearch | PostgreSQL | Redis | MongoDB | Translation Service |
|------------|---------------|------------|-------|---------|---------------------|
| Multi-Language Search | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Language Detection | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent |
| Translation | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Document Storage | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ✅ Excellent | ❌ Poor |
| Translation Cache | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor |

---

## Database Selection Framework (Continued)

### Additional Search System Patterns

**Pattern 1: Semantic Search**
- **Requirement**: Vector embeddings, similarity search
- **Databases**: Vector DB (Pinecone, Weaviate), Elasticsearch, PostgreSQL (pgvector)
- **Use Cases**: Semantic search, similarity search, recommendations

**Pattern 2: Image Search**
- **Requirement**: Visual similarity, embeddings, reverse search
- **Databases**: Vector DB, S3 (storage), Elasticsearch (metadata)
- **Use Cases**: Image search, reverse image search, visual similarity

**Pattern 3: Voice Search**
- **Requirement**: Speech-to-text, natural language, search
- **Databases**: Elasticsearch (search), S3 (audio), PostgreSQL (metadata)
- **Use Cases**: Voice search, voice assistants, voice queries

**Pattern 4: Multi-Language Search**
- **Requirement**: Multiple languages, translation, language detection
- **Databases**: Elasticsearch (multi-language), Translation APIs, Redis (cache)
- **Use Cases**: International search, multi-language content, translation

---

## Summary

**Problems Covered (66-70)**:
66. Enterprise Search System → Elasticsearch, PostgreSQL, S3, Redis, MongoDB
67. Semantic Search System → Vector DB, Elasticsearch, PostgreSQL, Redis, ML Feature Store
68. Image Search System → Vector DB, S3, Elasticsearch, PostgreSQL, Redis
69. Voice Search System → Elasticsearch, PostgreSQL, S3, Redis, Cassandra
70. Multi-Language Search System → Elasticsearch, PostgreSQL, Redis, MongoDB, Translation Service

**Key Patterns Identified**:
- **Semantic Search**: Vector databases for embeddings and similarity
- **Image Search**: Vector DB for visual similarity, S3 for storage
- **Voice Search**: Elasticsearch for transcript search, S3 for audio
- **Multi-Language**: Elasticsearch with language analyzers, Translation APIs
- **Enterprise Search**: Elasticsearch for full-text search and faceting

**Next**: Problems 71-75 (Additional Storage Systems)

