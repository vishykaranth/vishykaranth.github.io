# Top 100 System Design Problems - Quick Reference Summary

## Complete List with Primary Database Recommendations

### Problems 1-5: Social & Communication
1. Social Media Platform → **Neo4j** (Graph) + Cassandra + Redis
2. Chat Application → **Cassandra** + Redis + PostgreSQL
3. Video Conferencing → **Redis** + PostgreSQL + S3
4. Notification System → **Redis** + Cassandra + RabbitMQ
5. Friend Recommendation → **Neo4j** + Elasticsearch + Redis

### Problems 6-10: E-Commerce & Marketplace
6. E-Commerce Platform → **PostgreSQL** + Elasticsearch + Redis
7. Food Delivery → **PostgreSQL (PostGIS)** + Redis + MongoDB
8. Ride-Sharing → **Redis** + PostgreSQL + TimescaleDB
9. Hotel Booking → **PostgreSQL** + Elasticsearch + Redis
10. Stock Trading → **PostgreSQL** + Redis + TimescaleDB

### Problems 11-15: Content & Media
11. Video Streaming → **Cassandra** + S3 + Elasticsearch
12. Music Streaming → **PostgreSQL** + S3 + Redis
13. News Feed → **Redis** + Cassandra + Neo4j
14. Blogging Platform → **PostgreSQL** + Elasticsearch + Redis
15. Photo Sharing → **Cassandra** + S3 + Redis

### Problems 16-20: Search & Discovery
16. Web Search Engine → **Elasticsearch** + BigTable + Cassandra
17. Distributed Search → **Elasticsearch** + PostgreSQL + Redis
18. Typeahead/Autocomplete → **Redis** + Elasticsearch
19. Product Search → **Elasticsearch** + PostgreSQL + Redis
20. Location-Based Search → **PostgreSQL (PostGIS)** + Elasticsearch

### Problems 21-25: Storage & File Systems
21. Distributed File System → **PostgreSQL** + HDFS + Redis
22. Cloud Storage → **PostgreSQL** + S3 + Redis
23. Distributed Cache → **Redis Cluster** + PostgreSQL
24. Key-Value Store → **DynamoDB** + Redis
25. Distributed Database → **Cassandra** + PostgreSQL + Redis

### Problems 26-30: Analytics & Monitoring
26. Distributed Logging → **Elasticsearch** + Kafka + S3
27. Metrics Collection → **Prometheus** + TimescaleDB
28. Real-Time Analytics → **Kafka** + TimescaleDB + Redis
29. URL Shortener → **Redis** + PostgreSQL
30. Rate Limiter → **Redis** + PostgreSQL

### Problems 31-35: Real-Time & Gaming
31. Real-Time Multiplayer Game → **Redis** + PostgreSQL + Cassandra
32. Live Streaming Platform → **Redis** + PostgreSQL + S3
33. Collaborative Editor → **PostgreSQL** + Redis + Operational Transform
34. Real-Time Dashboard → **Redis** + TimescaleDB + Elasticsearch
35. Leaderboard System → **Redis** + PostgreSQL

### Problems 36-40: Payment & Financial
36. Payment Processing → **PostgreSQL** + Redis + Message Queue
37. Wallet System → **PostgreSQL** + Redis
38. Cryptocurrency Exchange → **PostgreSQL** + Redis + TimescaleDB
39. Billing System → **PostgreSQL** + TimescaleDB
40. Fraud Detection → **Redis** + PostgreSQL + ML Store

### Problems 41-45: Infrastructure & DevOps
41. Load Balancer → **Redis** (session) + Configuration DB
42. Service Discovery → **etcd/Consul** + PostgreSQL
43. Distributed Configuration → **etcd/Consul** + Git
44. Distributed Lock Service → **Redis/ZooKeeper**
45. Distributed Task Scheduler → **PostgreSQL** + Redis + Message Queue

### Problems 46-50: Specialized Systems
46. Distributed Counter → **Redis** + PostgreSQL
47. Distributed Pub-Sub → **Kafka** + Redis
48. Distributed Queue → **RabbitMQ/Kafka** + Redis
49. Distributed Tracing → **Elasticsearch** + Kafka
50. Distributed ID Generator → **Redis** + PostgreSQL

### Problems 51-55: Additional Social Systems
51. Social Network Graph → **Neo4j** + Cassandra
52. Messaging Queue System → **Kafka** + Redis
53. Activity Feed → **Redis** + Cassandra
54. User Profile System → **PostgreSQL** + Redis
55. Content Moderation → **PostgreSQL** + Elasticsearch + ML

### Problems 56-60: Additional E-Commerce
56. Shopping Cart → **Redis** + PostgreSQL
57. Inventory Management → **PostgreSQL** + Redis
58. Recommendation Engine → **Redis** + Feature Store + ML
59. Review System → **PostgreSQL** + Elasticsearch
60. Payment Gateway → **PostgreSQL** + Redis + External APIs

### Problems 61-65: Additional Content
61. Content Delivery Network → **S3/CDN** + Redis
62. Image Processing Service → **S3** + Queue + Processing
63. Video Transcoding → **S3** + Queue + Workers
64. Document Management → **PostgreSQL** + S3 + Elasticsearch
65. Media Library → **PostgreSQL** + S3 + Elasticsearch

### Problems 66-70: Additional Search
66. Enterprise Search → **Elasticsearch** + PostgreSQL
67. Semantic Search → **Vector DB** + Elasticsearch
68. Image Search → **Vector DB** + Elasticsearch + S3
69. Voice Search → **Elasticsearch** + Speech Processing
70. Multi-Language Search → **Elasticsearch** + Translation

### Problems 71-75: Additional Storage
71. Object Storage Service → **S3/Blob Storage**
72. Block Storage Service → **Block Storage** + Metadata DB
73. Backup System → **S3** + PostgreSQL + Scheduling
74. Archive System → **S3/Glacier** + PostgreSQL
75. Data Lake → **S3/HDFS** + Metadata Catalog

### Problems 76-80: Additional Analytics
76. Business Intelligence → **Data Warehouse** + OLAP
77. Data Pipeline → **Kafka** + Processing + Storage
78. Event Sourcing → **Event Store** + Snapshots
79. CQRS System → **Read DB** + Write DB
80. Data Warehouse → **Columnar DB** + ETL

### Problems 81-85: Additional Real-Time
81. Real-Time Collaboration → **Redis** + Operational Transform
82. Live Chat Support → **Redis** + PostgreSQL
83. Real-Time Notifications → **Redis** + WebSocket
84. Live Analytics Dashboard → **Redis** + TimescaleDB
85. Real-Time Monitoring → **Prometheus** + Grafana

### Problems 86-90: Additional Infrastructure
86. API Gateway → **Redis** + Configuration DB
87. Service Mesh → **Configuration** + Observability
88. Container Registry → **Object Storage** + Metadata DB
89. CI/CD Pipeline → **Git** + Artifact Storage
90. Infrastructure Monitoring → **Prometheus** + TimescaleDB

### Problems 91-95: Additional Specialized
91. Feature Flag System → **Redis** + PostgreSQL
92. A/B Testing Platform → **PostgreSQL** + Redis + Analytics
93. Experimentation Platform → **PostgreSQL** + Analytics
94. Configuration Service → **etcd/Consul** + Git
95. Secrets Management → **Vault** + Encryption

### Problems 96-100: Advanced Systems
96. Multi-Tenant SaaS → **PostgreSQL** + Redis + Isolation
97. Microservices Architecture → **Multiple DBs** + Service Mesh
98. Event-Driven Architecture → **Kafka** + Event Stores
99. Serverless Platform → **Managed Services** + Functions
100. Edge Computing Platform → **Distributed DBs** + Edge Nodes

---

## Database Selection Decision Tree

```
1. Data Model?
   Graph → Neo4j, ArangoDB
   Document → MongoDB, CouchDB
   Key-Value → Redis, DynamoDB
   Wide Column → Cassandra, HBase
   Time-Series → TimescaleDB, InfluxDB, Prometheus
   Relational → PostgreSQL, MySQL
   Search → Elasticsearch, Solr
   Vector → Pinecone, Weaviate

2. Consistency?
   Strong → PostgreSQL, MySQL, MongoDB (transactions)
   Eventual → Cassandra, DynamoDB, MongoDB (default)

3. Scale?
   < 1M users → PostgreSQL, MySQL
   1M-100M → PostgreSQL (replicas), MongoDB
   > 100M → Cassandra, DynamoDB, MongoDB (sharded)

4. Read/Write?
   Read-Heavy → Redis (cache), PostgreSQL (replicas)
   Write-Heavy → Cassandra, DynamoDB
   Balanced → PostgreSQL, MongoDB

5. Latency?
   < 1ms → Redis (in-memory)
   < 10ms → PostgreSQL, MongoDB (with cache)
   < 100ms → Most databases

6. Query Complexity?
   Simple → Redis, DynamoDB
   Moderate → MongoDB, Cassandra
   Complex → PostgreSQL, MySQL
```

---

## Database Categories by Use Case

### Transactional Systems
- **Primary**: PostgreSQL, MySQL, SQL Server
- **Use Cases**: Payments, orders, financial transactions
- **Key Features**: ACID, strong consistency, complex queries

### High-Volume Writes
- **Primary**: Cassandra, DynamoDB, Kafka
- **Use Cases**: Logs, events, time-series, analytics
- **Key Features**: Horizontal scaling, high throughput

### Real-Time Systems
- **Primary**: Redis, Memcached, DynamoDB
- **Use Cases**: Caching, sessions, real-time state
- **Key Features**: Low latency, high throughput

### Search Systems
- **Primary**: Elasticsearch, Solr, Algolia
- **Use Cases**: Full-text search, content search
- **Key Features**: Relevance ranking, aggregations

### Graph Systems
- **Primary**: Neo4j, ArangoDB, Amazon Neptune
- **Use Cases**: Social networks, recommendations
- **Key Features**: Graph traversal, relationship queries

### Time-Series Systems
- **Primary**: TimescaleDB, InfluxDB, Prometheus
- **Use Cases**: Metrics, monitoring, IoT
- **Key Features**: Time-optimized storage, efficient queries

### Media Storage
- **Primary**: S3, Azure Blob, Google Cloud Storage
- **Use Cases**: Videos, images, files
- **Key Features**: Cost-effective, scalable, CDN integration

---

This summary provides quick reference for all 100 problems. Detailed analysis with database selection criteria, architecture diagrams, and selection matrices are provided in Parts 1-20.

