# Top 100 System Design Problems with Database Selection Criteria - Part 3

## Problems 11-15: Content & Media Systems

### Problem 11: Design a Video Streaming Service (Netflix/YouTube)

#### Requirements
- **Scale**: 200M+ users, 1B+ videos, 1TB+ daily streaming
- **Features**: Video upload, transcoding, streaming, recommendations, playlists
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Videos, metadata, user data, watch history, recommendations
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Cassandra (NoSQL - Wide Column)**

**Why**:
- **Video Metadata**: High read throughput for video catalog
- **Watch History**: Time-series data, high write volume
- **Horizontal Scaling**: Linear scaling with nodes
- **Schema**: `videos_by_category`, `watch_history_by_user`
- **Use Case**: Video catalog, watch history, user data

**Secondary Databases**:

1. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Video files, thumbnails, transcoded versions
   - **Why**: Cost-effective, scalable, CDN integration
   - **Size**: Videos can be GBs, multiple formats per video

2. **PostgreSQL** (Relational)
   - **Use Case**: User accounts, subscriptions, payments
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, subscriptions, payments, profiles

3. **Redis** (In-Memory Cache)
   - **Use Case**: Popular videos cache, recommendations cache, session storage
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Popular videos (1 hour), recommendations (30 min)

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Video search, content discovery
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Video documents with metadata, transcripts

5. **MongoDB** (Document Store)
   - **Use Case**: Video metadata, playlists, nested structures
   - **Why**: Flexible schema, nested documents, good for metadata
   - **Schema**: Video documents with metadata, playlists

6. **Machine Learning Store** (Feature Store)
   - **Use Case**: Recommendation features, user embeddings
   - **Why**: Feature serving, model predictions
   - **Tools**: Redis, Cassandra, or specialized feature stores

**Database Architecture**:
```
Request → API Gateway
    ├── Cassandra → Video catalog, watch history
    ├── S3 → Video files, thumbnails
    ├── PostgreSQL → Users, subscriptions, payments
    ├── Redis → Popular videos cache, recommendations
    ├── Elasticsearch → Video search
    ├── MongoDB → Video metadata, playlists
    └── Feature Store → ML features, recommendations
```

**Selection Matrix**:
| Requirement | Cassandra | S3 | PostgreSQL | Redis | Elasticsearch |
|------------|-----------|----|------------|-------|---------------|
| Video Storage | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Video Catalog | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |
| Watch History | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor |
| Video Search | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent |
| Recommendations | ⚠️ Moderate | ❌ Poor | ❌ Poor | ✅ Excellent | ⚠️ Moderate |

---

### Problem 12: Design a Music Streaming Service (Spotify)

#### Requirements
- **Scale**: 400M+ users, 70M+ songs, 1B+ streams/day
- **Features**: Music streaming, playlists, recommendations, offline downloads
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Songs, playlists, user data, listening history, recommendations
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **Music Metadata**: Complex relationships (artists, albums, songs)
- **Playlists**: User playlists with relationships
- **ACID Transactions**: For subscriptions, payments
- **Schema**: Normalized relational schema
- **Use Case**: Music catalog, playlists, users, subscriptions

**Secondary Databases**:

1. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Audio files (MP3, FLAC, etc.)
   - **Why**: Cost-effective, scalable, CDN integration
   - **Size**: Songs typically 3-10MB, multiple formats

2. **Redis** (In-Memory Cache)
   - **Use Case**: Popular songs cache, playlists cache, recommendations
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Popular songs (1 hour), playlists (30 min)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Listening history, play counts, user activity
   - **Why**: High write throughput, time-series data
   - **Schema**: `listening_history_by_user`, `play_counts_by_song`

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Song search, artist search, content discovery
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Song documents with metadata, lyrics

5. **MongoDB** (Document Store)
   - **Use Case**: Playlist details, nested structures
   - **Why**: Flexible schema, nested playlists
   - **Schema**: Playlist documents with songs, metadata

**Database Architecture**:
```
Request → API Gateway
    ├── PostgreSQL → Music catalog, playlists, users, subscriptions
    ├── S3 → Audio files
    ├── Redis → Popular songs cache, playlists cache
    ├── Cassandra → Listening history, play counts
    ├── Elasticsearch → Song search
    └── MongoDB → Playlist details
```

**Selection Matrix**:
| Requirement | PostgreSQL | S3 | Redis | Cassandra | Elasticsearch |
|------------|------------|----|----|-----------|---------------|
| Music Catalog | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |
| Audio Storage | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Playlists | ✅ Excellent | ❌ Poor | ✅ Good | ❌ Poor | ❌ Poor |
| Listening History | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ❌ Poor |
| Song Search | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 13: Design a News Feed System

#### Requirements
- **Scale**: 1B+ users, 100M+ posts/day, 10B+ feed views/day
- **Features**: Feed generation, ranking, personalization, real-time updates
- **Read/Write Ratio**: 1000:1 (reads dominate)
- **Data Types**: Posts, user data, social graph, engagement data
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Feed Cache**: Pre-computed feeds for users
- **Low Latency**: Sub-millisecond feed retrieval
- **High Throughput**: Millions of feed requests per second
- **Data Structures**: Sorted sets for ranked feeds
- **Use Case**: Cached feeds, hot posts, trending content

**Secondary Databases**:

1. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Post storage, user posts, feed generation source
   - **Why**: High write throughput, horizontal scaling
   - **Schema**: `posts_by_user`, `posts_by_timestamp`, `feed_source`

2. **PostgreSQL** (Relational)
   - **Use Case**: User accounts, relationships, metadata
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, relationships, settings

3. **Neo4j** (Graph Database)
   - **Use Case**: Social graph, follower relationships
   - **Why**: Efficient graph traversal, relationship queries
   - **Schema**: User nodes, follow relationships

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Content search, trending topics
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Post documents with content, metadata

5. **MongoDB** (Document Store)
   - **Use Case**: Post details, nested comments, engagement data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Post documents with comments, likes, shares

**Database Architecture**:
```
Feed Request → API Gateway
    ├── Redis → Cached feeds, hot posts
    ├── Cassandra → Post storage, feed source
    ├── PostgreSQL → Users, relationships
    ├── Neo4j → Social graph
    ├── Elasticsearch → Content search
    └── MongoDB → Post details, engagement
```

**Selection Matrix**:
| Requirement | Redis | Cassandra | PostgreSQL | Neo4j | Elasticsearch |
|------------|-------|-----------|------------|-------|---------------|
| Feed Retrieval | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Post Storage | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Social Graph | ❌ Poor | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ❌ Poor |
| Content Search | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor | ✅ Excellent |
| User Data | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor |

---

### Problem 14: Design a Blogging Platform (Medium)

#### Requirements
- **Scale**: 100M+ users, 10M+ articles, 1B+ reads/month
- **Features**: Article creation, publishing, reading, comments, recommendations
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Articles, users, comments, tags, reading history
- **Consistency**: Strong for publishing, eventual for recommendations

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for publishing, payments
- **Complex Queries**: Article filtering, relationships, aggregations
- **Full-Text Search**: PostgreSQL full-text search capabilities
- **Schema**: Normalized relational schema
- **Use Case**: Articles, users, comments, tags, relationships

**Secondary Databases**:

1. **Elasticsearch** (Search Engine)
   - **Use Case**: Article search, content discovery, recommendations
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Article documents with content, metadata, tags

2. **Redis** (In-Memory Cache)
   - **Use Case**: Popular articles cache, reading history, session storage
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Popular articles (1 hour), sessions (24 hours)

3. **MongoDB** (Document Store)
   - **Use Case**: Article content, nested comments, drafts
   - **Why**: Flexible schema, nested structures, versioning
   - **Schema**: Article documents with content, comments, metadata

4. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Article images, media files
   - **Why**: Cost-effective, scalable, CDN integration
   - **Size**: Images typically KBs to MBs

5. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Reading history, analytics, engagement data
   - **Why**: High write throughput, time-series data
   - **Schema**: `reading_history_by_user`, `engagement_by_article`

**Database Architecture**:
```
Request → API Gateway
    ├── PostgreSQL → Articles, users, comments, tags
    ├── Elasticsearch → Article search, recommendations
    ├── Redis → Popular articles cache, reading history
    ├── MongoDB → Article content, drafts
    ├── S3 → Images, media files
    └── Cassandra → Reading history, analytics
```

**Selection Matrix**:
| Requirement | PostgreSQL | Elasticsearch | Redis | MongoDB | S3 |
|------------|------------|---------------|-------|---------|----|
| Article Storage | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ❌ Poor |
| Article Search | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Comments | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent | ❌ Poor |
| Popular Articles | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor |
| Media Storage | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 15: Design a Photo Sharing Service (Instagram)

#### Requirements
- **Scale**: 1B+ users, 100M+ photos/day, 1B+ likes/day
- **Features**: Photo upload, feed, stories, comments, likes, search
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Photos, metadata, user data, social graph, engagement
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Cassandra (NoSQL - Wide Column)**

**Why**:
- **Photo Metadata**: High read throughput for photo feed
- **Horizontal Scaling**: Linear scaling with nodes
- **Time-Series Data**: Photos are time-ordered
- **Schema**: `photos_by_user`, `feed_by_user`, `photos_by_timestamp`
- **Use Case**: Photo feed, metadata, engagement data

**Secondary Databases**:

1. **S3/Blob Storage** (Object Storage)
   - **Use Case**: Photo files, thumbnails, processed images
   - **Why**: Cost-effective, scalable, CDN integration
   - **Size**: Photos typically 1-5MB, multiple sizes per photo

2. **Redis** (In-Memory Cache)
   - **Use Case**: Feed cache, popular photos, session storage
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Feed cache (5 min), popular photos (1 hour)

3. **PostgreSQL** (Relational)
   - **Use Case**: User accounts, relationships, settings
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Users, relationships, settings, metadata

4. **Neo4j** (Graph Database)
   - **Use Case**: Social graph, follower relationships
   - **Why**: Efficient graph traversal, relationship queries
   - **Schema**: User nodes, follow relationships

5. **MongoDB** (Document Store)
   - **Use Case**: Photo details, nested comments, engagement data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Photo documents with comments, likes, metadata

6. **Elasticsearch** (Search Engine)
   - **Use Case**: User search, hashtag search, location search
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: User documents, hashtag documents, location data

**Database Architecture**:
```
Request → API Gateway
    ├── Cassandra → Photo feed, metadata, engagement
    ├── S3 → Photo files, thumbnails
    ├── Redis → Feed cache, popular photos
    ├── PostgreSQL → Users, relationships
    ├── Neo4j → Social graph
    ├── MongoDB → Photo details, comments
    └── Elasticsearch → User search, hashtag search
```

**Selection Matrix**:
| Requirement | Cassandra | S3 | Redis | PostgreSQL | Neo4j |
|------------|-----------|----|----|------------|-------|
| Photo Storage | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Photo Feed | ✅ Excellent | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor |
| Social Graph | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate | ✅ Excellent |
| User Search | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate | ❌ Poor |
| Engagement Data | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor |

---

## Database Selection Framework (Continued)

### Content & Media Patterns

**Pattern 1: Media Storage**
- **Requirement**: Large files (videos, images, audio)
- **Databases**: S3, Azure Blob, Google Cloud Storage
- **Use Cases**: Video streaming, photo sharing, music streaming

**Pattern 2: Content Catalog**
- **Requirement**: High read throughput, metadata storage
- **Databases**: Cassandra, MongoDB, PostgreSQL
- **Use Cases**: Video catalog, music catalog, photo feed

**Pattern 3: Content Search**
- **Requirement**: Full-text search, relevance ranking
- **Databases**: Elasticsearch, Solr, Algolia
- **Use Cases**: Video search, article search, content discovery

**Pattern 4: Content Recommendations**
- **Requirement**: ML features, user embeddings
- **Databases**: Redis (cache), Feature stores, Vector databases
- **Use Cases**: Video recommendations, music recommendations

---

## Summary

**Problems Covered (11-15)**:
11. Video Streaming Service → Cassandra, S3, PostgreSQL, Redis, Elasticsearch
12. Music Streaming Service → PostgreSQL, S3, Redis, Cassandra, Elasticsearch
13. News Feed System → Redis, Cassandra, PostgreSQL, Neo4j, Elasticsearch
14. Blogging Platform → PostgreSQL, Elasticsearch, Redis, MongoDB, S3
15. Photo Sharing Service → Cassandra, S3, Redis, PostgreSQL, Neo4j

**Key Patterns Identified**:
- **Media Storage**: S3/Blob storage for large files
- **Content Catalog**: Cassandra/MongoDB for high read throughput
- **Content Search**: Elasticsearch for full-text search
- **Feed Systems**: Redis for cached feeds
- **Social Graph**: Neo4j for relationship queries

**Next**: Problems 16-20 (Search & Discovery Systems)

