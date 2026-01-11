# 50 System Design Problems - Comprehensive Collection

## Overview

This document contains 50 diverse system design problems that test different aspects of system design skills, including scalability, consistency, caching, real-time processing, distributed systems, and more.

---

## Category 1: Social & Communication Systems

### 1. Design a Social Media Platform (Facebook/Twitter)
**Focus Areas**: News feed, timeline generation, social graph, real-time updates
**Key Challenges**: 
- Feed ranking algorithms
- Graph database for relationships
- Real-time notifications
- Content moderation at scale

### 2. Design a Chat Application (WhatsApp/Slack)
**Focus Areas**: Real-time messaging, message delivery, group chats, presence
**Key Challenges**:
- WebSocket connection management
- Message ordering and delivery guarantees
- Offline message sync
- End-to-end encryption

### 3. Design a Video Conferencing System (Zoom/Teams)
**Focus Areas**: Video/audio streaming, screen sharing, recording, low latency
**Key Challenges**:
- Real-time media streaming
- Bandwidth optimization
- Multi-party synchronization
- Recording and playback

### 4. Design a Notification System
**Focus Areas**: Multi-channel delivery, queuing, batching, rate limiting
**Key Challenges**:
- Push notifications (iOS/Android/Web)
- Email/SMS delivery
- Notification preferences
- Delivery tracking

### 5. Design a Friend Recommendation System
**Focus Areas**: Graph algorithms, machine learning, real-time processing
**Key Challenges**:
- Graph traversal at scale
- Feature engineering
- Model training and serving
- Real-time recommendations

---

## Category 2: E-Commerce & Marketplace

### 6. Design an E-Commerce Platform (Amazon)
**Focus Areas**: Product catalog, search, shopping cart, checkout, inventory
**Key Challenges**:
- Product search and filtering
- Inventory management
- Order processing pipeline
- Payment integration

### 7. Design a Food Delivery System (Uber Eats/DoorDash)
**Focus Areas**: Order matching, real-time tracking, driver assignment, routing
**Key Challenges**:
- Real-time location tracking
- Driver-rider matching algorithm
- ETA calculation
- Dynamic pricing

### 8. Design a Ride-Sharing System (Uber/Lyft)
**Focus Areas**: Driver-passenger matching, real-time tracking, surge pricing, payments
**Key Challenges**:
- Geospatial indexing
- Real-time matching algorithm
- Dynamic pricing
- Trip tracking

### 9. Design a Hotel Booking System (Booking.com)
**Focus Areas**: Search, availability, pricing, reservations, reviews
**Key Challenges**:
- Complex search queries
- Availability management
- Price aggregation
- Booking conflicts

### 10. Design a Stock Trading Platform
**Focus Areas**: Order matching, real-time quotes, portfolio management, risk
**Key Challenges**:
- Low-latency order processing
- Real-time market data
- Order book management
- Compliance and audit

---

## Category 3: Content & Media

### 11. Design a Video Streaming Service (Netflix/YouTube)
**Focus Areas**: Video storage, CDN, transcoding, recommendations, playback
**Key Challenges**:
- Video encoding/transcoding
- CDN distribution
- Adaptive bitrate streaming
- Recommendation engine

### 12. Design a Music Streaming Service (Spotify)
**Focus Areas**: Audio streaming, playlist management, recommendations, offline
**Key Challenges**:
- Audio streaming optimization
- Playlist synchronization
- Music recommendation
- Offline downloads

### 13. Design a News Feed System
**Focus Areas**: Content aggregation, ranking, personalization, real-time updates
**Key Challenges**:
- Content ranking algorithms
- Real-time feed generation
- Personalization
- Content moderation

### 14. Design a Blogging Platform (Medium)
**Focus Areas**: Content creation, publishing, reading, comments, recommendations
**Key Challenges**:
- Rich text editor
- Content versioning
- Comment threading
- Reading time estimation

### 15. Design a Photo Sharing Service (Instagram)
**Focus Areas**: Image storage, processing, feed, stories, search
**Key Challenges**:
- Image processing pipeline
- Feed generation
- Stories (24-hour content)
- Image search

---

## Category 4: Search & Discovery

### 16. Design a Web Search Engine (Google)
**Focus Areas**: Web crawling, indexing, ranking, query processing
**Key Challenges**:
- Distributed web crawling
- Inverted index
- PageRank algorithm
- Query understanding

### 17. Design a Distributed Search System (Elasticsearch)
**Focus Areas**: Distributed indexing, query routing, sharding, replication
**Key Challenges**:
- Distributed indexing
- Query coordination
- Shard management
- Consistency models

### 18. Design a Typeahead/Autocomplete System
**Focus Areas**: Prefix matching, ranking, real-time updates, caching
**Key Challenges**:
- Trie data structure
- Real-time updates
- Ranking suggestions
- Caching strategies

### 19. Design a Product Search for E-Commerce
**Focus Areas**: Faceted search, filtering, sorting, relevance ranking
**Key Challenges**:
- Multi-faceted filtering
- Relevance scoring
- Search result ranking
- Spell correction

### 20. Design a Location-Based Search (Yelp)
**Focus Areas**: Geospatial search, ranking, reviews, recommendations
**Key Challenges**:
- Geospatial indexing (R-tree, Geohash)
- Distance calculation
- Review aggregation
- Business recommendations

---

## Category 5: Storage & File Systems

### 21. Design a Distributed File System (Google File System)
**Focus Areas**: File storage, replication, consistency, fault tolerance
**Key Challenges**:
- Distributed storage
- Replication strategy
- Consistency guarantees
- Fault tolerance

### 22. Design a Cloud Storage Service (Dropbox/Google Drive)
**Focus Areas**: File upload/download, sync, versioning, sharing
**Key Challenges**:
- File synchronization
- Conflict resolution
- Version control
- Sharing and permissions

### 23. Design a Distributed Cache (Redis Cluster)
**Focus Areas**: Sharding, replication, consistency, eviction
**Key Challenges**:
- Consistent hashing
- Replication strategy
- Cache invalidation
- Eviction policies

### 24. Design a Key-Value Store (DynamoDB)
**Focus Areas**: Partitioning, replication, consistency, availability
**Key Challenges**:
- Consistent hashing
- Vector clocks
- Eventual consistency
- Conflict resolution

### 25. Design a Distributed Database (Cassandra)
**Focus Areas**: Partitioning, replication, consistency, write/read paths
**Key Challenges**:
- Partitioning strategy
- Replication factor
- Consistency levels
- Hinted handoff

---

## Category 6: Analytics & Monitoring

### 26. Design a Distributed Logging System
**Focus Areas**: Log aggregation, storage, querying, real-time processing
**Key Challenges**:
- High-volume log ingestion
- Distributed storage
- Log querying
- Real-time alerting

### 27. Design a Metrics Collection System (Prometheus)
**Focus Areas**: Metrics collection, storage, querying, alerting
**Key Challenges**:
- Time-series database
- Metric aggregation
- Query language
- Alerting rules

### 28. Design a Real-Time Analytics System
**Focus Areas**: Stream processing, aggregation, dashboards, real-time queries
**Key Challenges**:
- Stream processing
- Real-time aggregation
- Sliding windows
- Low-latency queries

### 29. Design a URL Shortener (TinyURL/bit.ly)
**Focus Areas**: Short URL generation, redirect, analytics, scaling
**Key Challenges**:
- Base62 encoding
- Distributed ID generation
- High redirect rate
- Analytics tracking

### 30. Design a Rate Limiter
**Focus Areas**: Distributed rate limiting, sliding window, token bucket
**Key Challenges**:
- Distributed coordination
- Sliding window algorithm
- Token bucket algorithm
- High throughput

---

## Category 7: Real-Time & Gaming

### 31. Design a Real-Time Multiplayer Game Server
**Focus Areas**: Game state synchronization, low latency, anti-cheat, matchmaking
**Key Challenges**:
- State synchronization
- Lag compensation
- Anti-cheat mechanisms
- Matchmaking algorithm

### 32. Design a Live Streaming Platform (Twitch)
**Focus Areas**: Video streaming, chat, donations, moderation
**Key Challenges**:
- Low-latency streaming
- Real-time chat
- Payment processing
- Content moderation

### 33. Design a Collaborative Editor (Google Docs)
**Focus Areas**: Operational transformation, conflict resolution, real-time sync
**Key Challenges**:
- Operational transformation
- Conflict resolution
- Real-time synchronization
- Version control

### 34. Design a Real-Time Dashboard
**Focus Areas**: Real-time data updates, WebSocket, data aggregation
**Key Challenges**:
- Real-time data pipeline
- WebSocket management
- Data aggregation
- Dashboard rendering

### 35. Design a Leaderboard System
**Focus Areas**: Ranking, real-time updates, time-based leaderboards
**Key Challenges**:
- Efficient ranking
- Real-time updates
- Time-based leaderboards
- Pagination

---

## Category 8: Payment & Financial

### 36. Design a Payment Processing System (Stripe)
**Focus Areas**: Payment gateway, fraud detection, reconciliation, refunds
**Key Challenges**:
- Payment gateway integration
- Fraud detection
- Idempotency
- Reconciliation

### 37. Design a Wallet System (PayPal)
**Focus Areas**: Balance management, transactions, transfers, security
**Key Challenges**:
- ACID transactions
- Double-spending prevention
- Security and encryption
- Transaction history

### 38. Design a Cryptocurrency Exchange
**Focus Areas**: Order matching, order book, trading, wallet management
**Key Challenges**:
- Order matching engine
- Order book management
- Hot/cold wallet management
- Security

### 39. Design a Billing System
**Focus Areas**: Usage tracking, invoicing, payment processing, reporting
**Key Challenges**:
- Usage metering
- Invoice generation
- Payment processing
- Financial reporting

### 40. Design a Fraud Detection System
**Focus Areas**: Real-time detection, machine learning, rule engine, alerting
**Key Challenges**:
- Real-time processing
- ML model serving
- Rule engine
- False positive reduction

---

## Category 9: Infrastructure & DevOps

### 41. Design a Load Balancer
**Focus Areas**: Request routing, health checks, session affinity, SSL termination
**Key Challenges**:
- Load balancing algorithms
- Health checking
- Session persistence
- SSL/TLS handling

### 42. Design a Service Discovery System (Consul/etcd)
**Focus Areas**: Service registration, health checks, service discovery, configuration
**Key Challenges**:
- Service registration
- Health monitoring
- Service discovery
- Configuration management

### 43. Design a Distributed Configuration System
**Focus Areas**: Configuration storage, versioning, change notifications, rollback
**Key Challenges**:
- Configuration storage
- Version control
- Change notifications
- Rollback mechanism

### 44. Design a Distributed Lock Service
**Focus Areas**: Lock acquisition, expiration, fairness, deadlock prevention
**Key Challenges**:
- Distributed coordination
- Lock expiration
- Fairness guarantees
- Deadlock prevention

### 45. Design a Distributed Task Scheduler (Cron)
**Focus Areas**: Task scheduling, execution, retry, monitoring
**Key Challenges**:
- Distributed scheduling
- Task execution
- Retry mechanism
- Monitoring and alerting

---

## Category 10: Specialized Systems

### 46. Design a Distributed Counter
**Focus Areas**: Increment/decrement, consistency, sharding, aggregation
**Key Challenges**:
- Distributed increments
- Consistency guarantees
- Sharding strategy
- Aggregation

### 47. Design a Distributed Pub-Sub System (Kafka)
**Focus Areas**: Message publishing, subscription, partitioning, replication
**Key Challenges**:
- Topic partitioning
- Consumer groups
- Message ordering
- Replication

### 48. Design a Distributed Queue (SQS)
**Focus Areas**: Message queuing, delivery guarantees, visibility timeout, dead-letter
**Key Challenges**:
- Message delivery
- At-least-once delivery
- Visibility timeout
- Dead-letter queues

### 49. Design a Distributed Tracing System (Jaeger/Zipkin)
**Focus Areas**: Trace collection, storage, querying, visualization
**Key Challenges**:
- Trace collection
- Span correlation
- Storage optimization
- Query performance

### 50. Design a Distributed ID Generator (Snowflake)
**Focus Areas**: Unique ID generation, scalability, time-based ordering
**Key Challenges**:
- Uniqueness guarantees
- Scalability
- Time-based ordering
- Clock synchronization

---

## Design Problem Template

For each problem, consider:

### 1. Requirements Gathering
- **Functional Requirements**: What the system should do
- **Non-Functional Requirements**: Performance, scalability, availability
- **Scale**: Users, requests per second, data volume
- **Constraints**: Latency, consistency, cost

### 2. High-Level Design
- **System Architecture**: Components and their interactions
- **API Design**: REST/GraphQL/gRPC endpoints
- **Database Design**: Schema, indexes, partitioning
- **Caching Strategy**: What to cache, eviction policies
- **Load Balancing**: How to distribute load

### 3. Detailed Design
- **Data Models**: Entities and relationships
- **Algorithms**: Core algorithms (matching, ranking, etc.)
- **Data Flow**: Request/response flow
- **Error Handling**: Failure scenarios and recovery
- **Security**: Authentication, authorization, encryption

### 4. Scalability & Performance
- **Horizontal Scaling**: How to scale out
- **Vertical Scaling**: When to scale up
- **Database Scaling**: Read replicas, sharding
- **Caching**: Multi-level caching
- **CDN**: Content delivery

### 5. Reliability & Availability
- **Fault Tolerance**: Handling failures
- **Replication**: Data replication strategy
- **Backup & Recovery**: Disaster recovery
- **Monitoring**: Metrics and alerting

---

## Key System Design Concepts by Problem

### Scalability
- Problems: 1, 6, 11, 16, 21, 26, 31, 36, 41, 46

### Consistency
- Problems: 22, 24, 25, 33, 37, 46

### Real-Time Processing
- Problems: 2, 3, 12, 28, 31, 32, 34, 40

### Distributed Systems
- Problems: 17, 21, 24, 25, 42, 47, 48, 49

### Caching
- Problems: 1, 6, 11, 18, 23, 29

### Search & Ranking
- Problems: 1, 16, 17, 19, 20, 35

### Message Queues
- Problems: 4, 27, 47, 48

### Geospatial
- Problems: 7, 8, 9, 20

### Media Processing
- Problems: 11, 12, 15, 32

### Financial Systems
- Problems: 10, 36, 37, 38, 39, 40

---

## Recommended Study Order

### Beginner Level
1. URL Shortener (29)
2. Rate Limiter (30)
3. Distributed Counter (46)
4. Distributed ID Generator (50)
5. Leaderboard System (35)

### Intermediate Level
1. Chat Application (2)
2. News Feed System (13)
3. Typeahead System (18)
4. Distributed Cache (23)
5. Load Balancer (41)

### Advanced Level
1. Web Search Engine (16)
2. Distributed File System (21)
3. Distributed Database (25)
4. Real-Time Multiplayer Game (31)
5. Collaborative Editor (33)

### Expert Level
1. Social Media Platform (1)
2. Video Streaming Service (11)
3. Payment Processing System (36)
4. Distributed Pub-Sub System (47)
5. Distributed Tracing System (49)

---

## Tips for System Design Interviews

1. **Clarify Requirements**: Ask about scale, constraints, and priorities
2. **Start High-Level**: Draw architecture diagram first
3. **Think Scalable**: Design for 10x, 100x scale
4. **Consider Trade-offs**: Consistency vs Availability, Latency vs Throughput
5. **Discuss Alternatives**: Mention different approaches and their trade-offs
6. **Estimate Capacity**: Back-of-envelope calculations
7. **Handle Failures**: Discuss failure scenarios and recovery
8. **Security**: Authentication, authorization, encryption
9. **Monitoring**: Metrics, logging, alerting
10. **Iterate**: Refine design based on feedback

---

## Additional Resources

- **Books**: 
  - "Designing Data-Intensive Applications" by Martin Kleppmann
  - "System Design Interview" by Alex Xu

- **Online Platforms**:
  - LeetCode System Design
  - Educative.io System Design Course
  - Grokking the System Design Interview

- **Practice**:
  - Design 2-3 problems per week
  - Focus on different categories
  - Time yourself (45-60 minutes)
  - Get feedback from peers

---

## Summary

These 50 problems cover:
- **10 Categories**: Social, E-commerce, Media, Search, Storage, Analytics, Real-time, Payment, Infrastructure, Specialized
- **Key Concepts**: Scalability, Consistency, Caching, Distributed Systems, Real-time Processing
- **Difficulty Levels**: Beginner to Expert
- **Real-World Systems**: Based on actual production systems

Practice these problems to master system design skills across different domains and scenarios.

