# Top 100 System Design Problems with Database Selection Criteria - Part 13

## Problems 61-65: Additional Content Systems

### Problem 61: Design a Content Delivery Network (CDN)

#### Requirements
- **Scale**: 1B+ requests/day, petabytes of content, global distribution
- **Features**: Content caching, edge locations, cache invalidation, analytics
- **Read/Write Ratio**: 10000:1 (reads dominate)
- **Data Types**: Content, cache metadata, analytics, configurations
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: S3/Blob Storage (Object Storage)**

**Why**:
- **Content Storage**: Origin storage for CDN content
- **Cost-Effective**: Low-cost storage for large files
- **Scalability**: Unlimited storage capacity
- **CDN Integration**: Direct integration with CDN providers
- **Use Case**: Origin content storage, static assets, media files

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Cache invalidation tags, cache metadata, analytics
   - **Why**: Sub-millisecond latency, high throughput
   - **TTL**: Cache tags (1 hour), metadata (5 min)

2. **PostgreSQL** (Relational)
   - **Use Case**: CDN configuration, cache rules, analytics metadata
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: CDN configurations, cache rules, analytics

3. **Elasticsearch** (Search Engine)
   - **Use Case**: Content search, analytics, log analysis
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Content metadata, access logs, analytics

4. **TimescaleDB** (Time-Series)
   - **Use Case**: CDN metrics, bandwidth metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: CDN metrics over time, bandwidth usage

5. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Access logs, analytics data, high-scale storage
   - **Why**: High write throughput, time-series data
   - **Schema**: `access_logs_by_timestamp`, `analytics_by_content`

**Database Architecture**:
```
Content Request → CDN Edge → Origin
    ├── S3/Blob Storage → Origin content storage
    ├── Redis → Cache invalidation, metadata, analytics
    ├── PostgreSQL → CDN configuration, cache rules
    ├── Elasticsearch → Content search, analytics
    ├── TimescaleDB → CDN metrics, bandwidth metrics
    └── Cassandra → Access logs, analytics data
```

**Selection Matrix**:
| Requirement | S3 | Redis | PostgreSQL | Elasticsearch | TimescaleDB |
|------------|----|----|------------|---------------|-------------|
| Content Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Cache Invalidation | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| CDN Configuration | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor |
| Analytics | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |
| Access Logs | ❌ Poor | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |

---

### Problem 62: Design an Image Processing Service

#### Requirements
- **Scale**: 100M+ images/day, real-time processing, multiple formats
- **Features**: Image upload, resizing, cropping, format conversion, optimization
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Images, metadata, processing jobs, results
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: S3/Blob Storage (Object Storage)**

**Why**:
- **Image Storage**: Original and processed images
- **Cost-Effective**: Low-cost storage for large files
- **Scalability**: Unlimited storage capacity
- **CDN Integration**: Direct CDN integration
- **Use Case**: Image storage, processed images, thumbnails

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Image metadata, processing jobs, results
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Images, processing jobs, results, metadata

2. **Redis** (In-Memory Cache)
   - **Use Case**: Processing queue, job status, recent images cache
   - **Why**: Sub-millisecond latency, high throughput, queues
   - **TTL**: Job status (1 hour), recent images (30 min)

3. **Message Queue (RabbitMQ/Kafka)**
   - **Use Case**: Image processing queue, async processing
   - **Why**: Reliable delivery, multiple workers, retry
   - **Pattern**: Producer-consumer for image processing

4. **MongoDB** (Document Store)
   - **Use Case**: Image metadata, nested processing data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Image documents with metadata, processing results

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Processing metrics, performance data, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Processing metrics over time, performance data

**Database Architecture**:
```
Image Upload → Image Service
    ├── S3 → Image storage, processed images
    ├── PostgreSQL → Image metadata, processing jobs
    ├── Redis → Processing queue, job status
    ├── Message Queue → Image processing queue
    ├── MongoDB → Image metadata, nested data
    └── TimescaleDB → Processing metrics, analytics
```

**Selection Matrix**:
| Requirement | S3 | PostgreSQL | Redis | Message Queue | MongoDB |
|------------|----|------------|-------|---------------|---------|
| Image Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Processing Jobs | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Processing Queue | ❌ Poor | ❌ Poor | ✅ Excellent | ✅ Excellent | ❌ Poor |
| Image Metadata | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent |
| Processing Metrics | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |

---

### Problem 63: Design a Video Transcoding Service

#### Requirements
- **Scale**: 10M+ videos/day, multiple formats, real-time transcoding
- **Features**: Video upload, transcoding, format conversion, quality optimization
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Videos, transcoding jobs, formats, metadata
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: S3/Blob Storage (Object Storage)**

**Why**:
- **Video Storage**: Original and transcoded videos
- **Cost-Effective**: Low-cost storage for large files
- **Scalability**: Unlimited storage capacity
- **CDN Integration**: Direct CDN integration
- **Use Case**: Video storage, transcoded videos, multiple formats

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Video metadata, transcoding jobs, formats
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Videos, transcoding jobs, formats, metadata

2. **Redis** (In-Memory Cache)
   - **Use Case**: Transcoding queue, job status, recent videos cache
   - **Why**: Sub-millisecond latency, high throughput, queues
   - **TTL**: Job status (24 hours), recent videos (1 hour)

3. **Message Queue (RabbitMQ/Kafka)**
   - **Use Case**: Video transcoding queue, async processing
   - **Why**: Reliable delivery, multiple workers, retry
   - **Pattern**: Producer-consumer for video transcoding

4. **MongoDB** (Document Store)
   - **Use Case**: Video metadata, nested transcoding data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Video documents with metadata, transcoding results

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Transcoding metrics, performance data, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Transcoding metrics over time, performance data

**Database Architecture**:
```
Video Upload → Transcoding Service
    ├── S3 → Video storage, transcoded videos
    ├── PostgreSQL → Video metadata, transcoding jobs
    ├── Redis → Transcoding queue, job status
    ├── Message Queue → Video transcoding queue
    ├── MongoDB → Video metadata, nested data
    └── TimescaleDB → Transcoding metrics, analytics
```

**Selection Matrix**:
| Requirement | S3 | PostgreSQL | Redis | Message Queue | MongoDB |
|------------|----|------------|-------|---------------|---------|
| Video Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Transcoding Jobs | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |
| Transcoding Queue | ❌ Poor | ❌ Poor | ✅ Excellent | ✅ Excellent | ❌ Poor |
| Video Metadata | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent |
| Transcoding Metrics | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |

---

### Problem 64: Design a Document Management System

#### Requirements
- **Scale**: 100M+ documents, 1M+ documents/day, search, versioning
- **Features**: Document storage, search, versioning, collaboration, permissions
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Documents, versions, metadata, permissions, search index
- **Consistency**: Strong consistency for permissions

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for document operations, permissions
- **Complex Queries**: Document relationships, permissions, versioning
- **Full-Text Search**: PostgreSQL full-text search capabilities
- **Relationships**: Documents, versions, users, permissions
- **Use Case**: Documents, versions, permissions, metadata

**Secondary Databases**:

1. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Document file storage, versions
   - **Why**: Cost-effective, scalable, versioning support
   - **Size**: Documents can be MBs to GBs

2. **Elasticsearch** (Search Engine)
   - **Use Case**: Document search, content search, full-text search
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Document content indexed for search

3. **Redis** (In-Memory Cache)
   - **Use Case**: Recent documents cache, frequently accessed documents
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Recent documents (1 hour), frequently accessed (24 hours)

4. **MongoDB** (Document Store)
   - **Use Case**: Document metadata, nested document data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Document documents with metadata, versions

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Document access metrics, version history, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Document access over time, version history

**Database Architecture**:
```
Document Operation → Document Service
    ├── PostgreSQL → Documents, versions, permissions (ACID)
    ├── S3 → Document file storage, versions
    ├── Elasticsearch → Document search, content search
    ├── Redis → Recent documents cache, frequently accessed
    ├── MongoDB → Document metadata, nested data
    └── TimescaleDB → Document access metrics, analytics
```

**Selection Matrix**:
| Requirement | PostgreSQL | S3 | Elasticsearch | Redis | MongoDB |
|------------|------------|----|---------------|-------|---------|
| Document Storage | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent |
| Document Search | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Version Control | ✅ Excellent | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent |
| Permissions | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Document Analytics | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ❌ Poor | ⚠️ Moderate |

---

### Problem 65: Design a Media Library System

#### Requirements
- **Scale**: 1B+ media files, 10M+ files/day, multiple formats, search
- **Features**: Media storage, search, metadata, transcoding, CDN integration
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Media files, metadata, formats, search index, analytics
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: S3/Blob Storage (Object Storage)**

**Why**:
- **Media Storage**: Original and processed media files
- **Cost-Effective**: Low-cost storage for large files
- **Scalability**: Unlimited storage capacity
- **CDN Integration**: Direct CDN integration
- **Use Case**: Media file storage, processed media, multiple formats

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Media metadata, formats, relationships
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Media files, metadata, formats, relationships

2. **Elasticsearch** (Search Engine)
   - **Use Case**: Media search, metadata search, content discovery
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Media documents with metadata, content

3. **Redis** (In-Memory Cache)
   - **Use Case**: Popular media cache, frequently accessed media, metadata cache
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Popular media (1 hour), metadata (30 min)

4. **MongoDB** (Document Store)
   - **Use Case**: Media metadata, nested metadata, flexible schema
   - **Why**: Flexible schema, nested structures
   - **Schema**: Media documents with metadata, formats

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Media access metrics, usage analytics, trends
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Media access over time, usage metrics

**Database Architecture**:
```
Media Request → Media Service
    ├── S3 → Media file storage, processed media
    ├── PostgreSQL → Media metadata, formats, relationships
    ├── Elasticsearch → Media search, metadata search
    ├── Redis → Popular media cache, metadata cache
    ├── MongoDB → Media metadata, nested data
    └── TimescaleDB → Media access metrics, analytics
```

**Selection Matrix**:
| Requirement | S3 | PostgreSQL | Elasticsearch | Redis | MongoDB |
|------------|----|------------|---------------|-------|---------|
| Media Storage | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Media Search | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate |
| Media Metadata | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Media Cache | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Media Analytics | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate |

---

## Database Selection Framework (Continued)

### Additional Content System Patterns

**Pattern 1: Media Storage**
- **Requirement**: Large files, cost-effective, scalable
- **Databases**: S3, Azure Blob, Google Cloud Storage
- **Use Cases**: Images, videos, documents, media files

**Pattern 2: Media Processing**
- **Requirement**: Processing queues, job management, metadata
- **Databases**: PostgreSQL (jobs), Message Queues (processing), S3 (storage)
- **Use Cases**: Image processing, video transcoding, document processing

**Pattern 3: Media Search**
- **Requirement**: Full-text search, metadata search, content discovery
- **Databases**: Elasticsearch, PostgreSQL (full-text)
- **Use Cases**: Media search, content discovery, metadata search

**Pattern 4: Media Analytics**
- **Requirement**: Access metrics, usage analytics, trends
- **Databases**: TimescaleDB, Elasticsearch, PostgreSQL
- **Use Cases**: Media analytics, usage tracking, performance metrics

---

## Summary

**Problems Covered (61-65)**:
61. Content Delivery Network → S3, Redis, PostgreSQL, Elasticsearch, TimescaleDB
62. Image Processing Service → S3, PostgreSQL, Redis, Message Queue, MongoDB
63. Video Transcoding Service → S3, PostgreSQL, Redis, Message Queue, MongoDB
64. Document Management System → PostgreSQL, S3, Elasticsearch, Redis, MongoDB
65. Media Library System → S3, PostgreSQL, Elasticsearch, Redis, MongoDB

**Key Patterns Identified**:
- **Media Storage**: S3/Blob storage for large files
- **Media Processing**: Message Queues for async processing
- **Media Search**: Elasticsearch for full-text search
- **Media Metadata**: PostgreSQL for complex queries
- **Media Analytics**: TimescaleDB for time-series analytics

**Next**: Problems 66-70 (Additional Search Systems)

