# Top 100 System Design Problems with Database Selection Criteria - Part 2

## Problems 6-10: E-Commerce & Marketplace Systems

### Problem 6: Design an E-Commerce Platform (Amazon)

#### Requirements
- **Scale**: 300M+ users, 1B+ products, 10M+ orders/day
- **Features**: Product catalog, search, shopping cart, checkout, inventory, recommendations
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Products, orders, users, inventory, reviews
- **Consistency**: Strong for orders/inventory, eventual for recommendations

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for orders, payments, inventory
- **Complex Queries**: Product filtering, joins, aggregations
- **Relationships**: Products, categories, orders, users
- **Schema**: Normalized relational schema
- **Use Case**: Orders, inventory, user accounts, payments

**Secondary Databases**:

1. **Elasticsearch** (Search Engine)
   - **Use Case**: Product search, filtering, faceted search
   - **Why**: Full-text search, relevance ranking, aggregations
   - **Schema**: Product documents with all attributes

2. **Redis** (In-Memory Cache)
   - **Use Case**: Shopping cart, product cache, session storage
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Cart (24 hours), product cache (1 hour)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Product catalog (read-heavy), order history
   - **Why**: High read throughput, horizontal scaling
   - **Schema**: `products_by_category`, `orders_by_user`

4. **MongoDB** (Document Store)
   - **Use Case**: Product details, reviews, nested structures
   - **Why**: Flexible schema, nested documents, good for products
   - **Schema**: Product documents with variants, reviews

5. **DynamoDB** (NoSQL - Key-Value)
   - **Use Case**: Shopping cart, session data
   - **Why**: Managed service, auto-scaling, low latency
   - **Schema**: Cart by user_id, session by session_id

**Database Architecture**:
```
Request → API Gateway
    ├── PostgreSQL → Orders, inventory, users, payments
    ├── Elasticsearch → Product search
    ├── Redis → Cart, product cache, sessions
    ├── Cassandra → Product catalog, order history
    ├── MongoDB → Product details, reviews
    └── DynamoDB → Cart, sessions (alternative)
```

**Selection Matrix**:
| Requirement | PostgreSQL | Elasticsearch | Redis | Cassandra | MongoDB |
|------------|------------|---------------|-------|-----------|---------|
| ACID Transactions | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate |
| Product Search | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |
| Shopping Cart | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |
| Product Catalog | ✅ Good | ✅ Excellent | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |
| Order Management | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |

---

### Problem 7: Design a Food Delivery System (Uber Eats/DoorDash)

#### Requirements
- **Scale**: 50M+ users, 1M+ orders/day, 100K+ restaurants
- **Features**: Restaurant search, order placement, real-time tracking, driver matching
- **Read/Write Ratio**: 10:1 (more reads)
- **Data Types**: Restaurants, orders, locations, drivers, menus
- **Consistency**: Strong for orders, eventual for tracking

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for orders, payments
- **Geospatial Queries**: Location-based restaurant search
- **Complex Queries**: Order matching, driver assignment
- **Schema**: Restaurants, orders, drivers, users
- **Extensions**: PostGIS for geospatial data

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Active orders, driver locations, restaurant cache
   - **Why**: Real-time updates, low latency, pub/sub
   - **TTL**: Active orders (2 hours), restaurant cache (30 min)

2. **MongoDB** (Document Store)
   - **Use Case**: Restaurant menus, order details, nested structures
   - **Why**: Flexible schema, nested menus, order documents
   - **Schema**: Restaurant documents with menus, orders

3. **Elasticsearch** (Search Engine)
   - **Use Case**: Restaurant search, cuisine filtering
   - **Why**: Full-text search, location-based search, aggregations
   - **Schema**: Restaurant documents with location, cuisine

4. **TimescaleDB** (Time-Series)
   - **Use Case**: Order tracking, delivery time metrics
   - **Why**: Time-series optimization, analytics
   - **Schema**: Order status over time, delivery metrics

5. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Order history, user order history
   - **Why**: High write throughput, time-series data
   - **Schema**: `orders_by_user`, `orders_by_restaurant`

**Database Architecture**:
```
Order Request → API Gateway
    ├── PostgreSQL → Orders, restaurants, drivers (with PostGIS)
    ├── Redis → Active orders, driver locations, cache
    ├── MongoDB → Menus, order details
    ├── Elasticsearch → Restaurant search
    ├── TimescaleDB → Order tracking, metrics
    └── Cassandra → Order history
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | MongoDB | Elasticsearch | TimescaleDB |
|------------|------------|-------|---------|---------------|-------------|
| Geospatial Queries | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ✅ Good | ⚠️ Moderate |
| Real-Time Tracking | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Good |
| Order Management | ✅ Excellent | ⚠️ Moderate | ✅ Good | ❌ Poor | ⚠️ Moderate |
| Restaurant Search | ⚠️ Moderate | ❌ Poor | ⚠️ Moderate | ✅ Excellent | ❌ Poor |
| Menu Storage | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor |

---

### Problem 8: Design a Ride-Sharing System (Uber/Lyft)

#### Requirements
- **Scale**: 100M+ users, 10M+ rides/day, 5M+ drivers
- **Features**: Driver-passenger matching, real-time tracking, surge pricing, payments
- **Read/Write Ratio**: 1:1 (real-time updates)
- **Data Types**: Users, drivers, rides, locations, payments
- **Consistency**: Strong for payments, eventual for tracking

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Real-Time Location**: Driver/passenger locations updated frequently
- **Low Latency**: Sub-millisecond access for matching
- **Geospatial**: Redis GeoHash for location queries
- **Pub/Sub**: Real-time updates to drivers/passengers
- **Use Case**: Active rides, driver locations, matching queue

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: User accounts, ride history, payments
   - **Why**: ACID transactions, complex queries, relationships
   - **Extensions**: PostGIS for geospatial queries
   - **Schema**: Users, drivers, rides, payments

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Ride history, location history, analytics
   - **Why**: High write throughput, time-series data
   - **Schema**: `rides_by_user`, `rides_by_driver`, `location_history`

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Location tracking, ride metrics, surge pricing data
   - **Why**: Time-series optimization, analytics
   - **Schema**: Location points, ride metrics over time

4. **DynamoDB** (NoSQL - Key-Value)
   - **Use Case**: Active ride state, driver availability
   - **Why**: Managed service, auto-scaling, low latency
   - **Schema**: Active rides by ride_id, drivers by driver_id

5. **MongoDB** (Document Store)
   - **Use Case**: Ride details, nested trip data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Ride documents with route, stops, details

**Database Architecture**:
```
Ride Request → API Gateway
    ├── Redis → Active rides, driver locations, matching
    ├── PostgreSQL → Users, ride history, payments
    ├── Cassandra → Ride history, location logs
    ├── TimescaleDB → Location tracking, metrics
    ├── DynamoDB → Active ride state
    └── MongoDB → Ride details, trip data
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | TimescaleDB | DynamoDB |
|------------|-------|------------|-----------|-------------|----------|
| Real-Time Location | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Good | ✅ Good |
| Geospatial Queries | ✅ Good | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |
| Ride History | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate |
| Payment Processing | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Location Tracking | ✅ Good | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate |

---

### Problem 9: Design a Hotel Booking System (Booking.com)

#### Requirements
- **Scale**: 200M+ users, 1M+ bookings/day, 1M+ hotels
- **Features**: Hotel search, availability, pricing, reservations, reviews
- **Read/Write Ratio**: 100:1 (reads dominate)
- **Data Types**: Hotels, rooms, bookings, prices, reviews
- **Consistency**: Strong for bookings, eventual for search

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for bookings, inventory
- **Complex Queries**: Availability checks, pricing, filtering
- **Relationships**: Hotels, rooms, bookings, users
- **Schema**: Normalized relational schema
- **Use Case**: Bookings, inventory, users, payments

**Secondary Databases**:

1. **Elasticsearch** (Search Engine)
   - **Use Case**: Hotel search, filtering, faceted search
   - **Why**: Full-text search, location-based search, aggregations
   - **Schema**: Hotel documents with location, amenities, prices

2. **Redis** (In-Memory Cache)
   - **Use Case**: Availability cache, pricing cache, session storage
   - **Why**: Fast availability checks, pricing lookups
   - **TTL**: Availability (5 min), pricing (1 hour), sessions (24 hours)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Booking history, availability logs
   - **Why**: High write throughput, time-series data
   - **Schema**: `bookings_by_user`, `bookings_by_hotel`, `availability_by_hotel`

4. **MongoDB** (Document Store)
   - **Use Case**: Hotel details, room information, reviews
   - **Why**: Flexible schema, nested structures, good for hotels
   - **Schema**: Hotel documents with rooms, amenities, reviews

5. **TimescaleDB** (Time-Series)
   - **Use Case**: Pricing history, availability trends
   - **Why**: Time-series optimization, analytics
   - **Schema**: Price changes over time, availability patterns

**Database Architecture**:
```
Booking Request → API Gateway
    ├── PostgreSQL → Bookings, inventory, users, payments
    ├── Elasticsearch → Hotel search
    ├── Redis → Availability cache, pricing cache
    ├── Cassandra → Booking history, availability logs
    ├── MongoDB → Hotel details, reviews
    └── TimescaleDB → Pricing history, trends
```

**Selection Matrix**:
| Requirement | PostgreSQL | Elasticsearch | Redis | Cassandra | MongoDB |
|------------|------------|---------------|-------|-----------|---------|
| Booking Transactions | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Hotel Search | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Availability Checks | ✅ Good | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor |
| Booking History | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent | ⚠️ Moderate |
| Hotel Details | ✅ Good | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent |

---

### Problem 10: Design a Stock Trading Platform

#### Requirements
- **Scale**: 10M+ users, 1B+ trades/day, real-time quotes
- **Features**: Order matching, real-time quotes, portfolio, risk management
- **Read/Write Ratio**: 1:1 (high frequency)
- **Data Types**: Orders, trades, quotes, portfolios, positions
- **Consistency**: Strong consistency required (financial data)

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for financial transactions
- **Strong Consistency**: Required for orders, trades, balances
- **Complex Queries**: Portfolio calculations, risk analysis
- **Schema**: Normalized relational schema
- **Use Case**: Orders, trades, portfolios, accounts

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Real-time quotes, order book, active orders
   - **Why**: Sub-millisecond latency, high throughput
   - **Data Structures**: Sorted sets for order book, pub/sub for quotes

2. **TimescaleDB** (Time-Series)
   - **Use Case**: Historical prices, trade history, market data
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Price history, trade history, market data

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Trade history, audit logs, market data
   - **Why**: High write throughput, time-series data
   - **Schema**: `trades_by_user`, `trades_by_symbol`, `market_data`

4. **InfluxDB** (Time-Series)
   - **Use Case**: Real-time market data, tick data
   - **Why**: Optimized for time-series, high write throughput
   - **Schema**: Tick data, market metrics

5. **MongoDB** (Document Store)
   - **Use Case**: Portfolio snapshots, complex positions
   - **Why**: Flexible schema, nested structures
   - **Schema**: Portfolio documents with positions, holdings

**Database Architecture**:
```
Trade Request → API Gateway
    ├── PostgreSQL → Orders, trades, portfolios, accounts
    ├── Redis → Real-time quotes, order book, active orders
    ├── TimescaleDB → Historical prices, trade history
    ├── Cassandra → Trade history, audit logs
    ├── InfluxDB → Real-time market data, tick data
    └── MongoDB → Portfolio snapshots, positions
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | TimescaleDB | Cassandra | InfluxDB |
|------------|------------|-------|-------------|-----------|----------|
| ACID Transactions | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Real-Time Quotes | ❌ Poor | ✅ Excellent | ❌ Poor | ❌ Poor | ✅ Excellent |
| Order Matching | ✅ Excellent | ✅ Good | ❌ Poor | ❌ Poor | ❌ Poor |
| Historical Data | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ✅ Good | ✅ Excellent |
| Trade History | ✅ Excellent | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |

---

## Database Selection Framework (Continued)

### Use Case Patterns

**Pattern 1: Transactional Systems**
- **Requirement**: ACID transactions, strong consistency
- **Databases**: PostgreSQL, MySQL, SQL Server
- **Use Cases**: Payments, orders, financial transactions

**Pattern 2: High-Volume Writes**
- **Requirement**: Millions of writes per second
- **Databases**: Cassandra, DynamoDB, MongoDB (sharded)
- **Use Cases**: Logs, events, time-series data

**Pattern 3: Real-Time Data**
- **Requirement**: Sub-millisecond latency
- **Databases**: Redis, Memcached, DynamoDB
- **Use Cases**: Caching, sessions, real-time state

**Pattern 4: Search-Heavy**
- **Requirement**: Full-text search, relevance ranking
- **Databases**: Elasticsearch, Solr, Algolia
- **Use Cases**: Product search, content search, recommendations

**Pattern 5: Graph Data**
- **Requirement**: Relationship queries, graph traversal
- **Databases**: Neo4j, ArangoDB, Amazon Neptune
- **Use Cases**: Social networks, recommendations, fraud detection

---

## Summary

**Problems Covered (6-10)**:
6. E-Commerce Platform → PostgreSQL, Elasticsearch, Redis, Cassandra, MongoDB
7. Food Delivery System → PostgreSQL, Redis, MongoDB, Elasticsearch, TimescaleDB
8. Ride-Sharing System → Redis, PostgreSQL, Cassandra, TimescaleDB, DynamoDB
9. Hotel Booking System → PostgreSQL, Elasticsearch, Redis, Cassandra, MongoDB
10. Stock Trading Platform → PostgreSQL, Redis, TimescaleDB, Cassandra, InfluxDB

**Key Patterns Identified**:
- **Transactional Systems**: PostgreSQL for ACID guarantees
- **Real-Time Systems**: Redis for low latency
- **Search Systems**: Elasticsearch for full-text search
- **Time-Series**: TimescaleDB, InfluxDB for temporal data
- **High-Volume**: Cassandra for write-heavy workloads

**Next**: Problems 11-15 (Content & Media Systems)

