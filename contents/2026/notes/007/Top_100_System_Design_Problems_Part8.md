# Top 100 System Design Problems with Database Selection Criteria - Part 8

## Problems 36-40: Payment & Financial Systems

### Problem 36: Design a Payment Processing System (Stripe)

#### Requirements
- **Scale**: 1M+ transactions/day, $1B+ processed/month, < 500ms latency
- **Features**: Payment gateway, fraud detection, reconciliation, refunds, webhooks
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

### Problem 37: Design a Wallet System (PayPal)

#### Requirements
- **Scale**: 400M+ users, 1B+ transactions/month, < 200ms latency
- **Features**: Balance management, transfers, transactions, security
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Balances, transactions, users, security data
- **Consistency**: Strong consistency required (financial data)

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for balance updates, transfers
- **Strong Consistency**: Required for financial data
- **Double-Spending Prevention**: Transaction isolation
- **Complex Queries**: Balance calculations, transaction history
- **Use Case**: Wallets, balances, transactions, users

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Balance cache, recent transactions cache, session storage
   - **Why**: Sub-millisecond latency, high read throughput
   - **TTL**: Balance cache (1 min), recent transactions (5 min)

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Transaction history, audit logs
   - **Why**: High write throughput, time-series data
   - **Schema**: `transactions_by_user`, `transactions_by_timestamp`

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Transaction metrics, balance history, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Balance changes over time, transaction metrics

4. **MongoDB** (Document Store)
   - **Use Case**: Transaction details, nested transaction data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Transaction documents with details, metadata

**Database Architecture**:
```
Wallet Operation → API Gateway
    ├── PostgreSQL → Wallets, balances, transactions (ACID)
    ├── Redis → Balance cache, recent transactions
    ├── Cassandra → Transaction history, audit logs
    ├── TimescaleDB → Transaction metrics, balance history
    └── MongoDB → Transaction details, metadata
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | Cassandra | TimescaleDB | MongoDB |
|------------|------------|-------|-----------|-------------|---------|
| ACID Transactions | ✅ Excellent | ❌ Poor | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate |
| Balance Management | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Double-Spending Prevention | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Transaction History | ✅ Excellent | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent |
| Low Latency Reads | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ⚠️ Moderate |

---

### Problem 38: Design a Cryptocurrency Exchange

#### Requirements
- **Scale**: 10M+ users, 1B+ trades/day, < 10ms latency
- **Features**: Order matching, order book, trading, wallet management
- **Read/Write Ratio**: 1:1 (balanced, high frequency)
- **Data Types**: Orders, trades, order book, wallets, market data
- **Consistency**: Strong consistency required (financial data)

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for trades, order matching
- **Strong Consistency**: Required for order book, balances
- **Complex Queries**: Order matching, portfolio calculations
- **Audit Trail**: Complete trade history
- **Use Case**: Orders, trades, wallets, accounts

**Secondary Databases**:

1. **Redis** (In-Memory Cache)
   - **Use Case**: Order book, active orders, real-time prices
   - **Why**: Sub-millisecond latency, sorted sets for order book
   - **Data Structures**: Sorted sets (order book), hashes (prices)

2. **TimescaleDB** (Time-Series)
   - **Use Case**: Price history, trade history, market data
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Price ticks, trade history, market data

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Trade history, order history, audit logs
   - **Why**: High write throughput, time-series data
   - **Schema**: `trades_by_user`, `trades_by_symbol`, `orders_by_timestamp`

4. **InfluxDB** (Time-Series)
   - **Use Case**: Real-time market data, tick data, metrics
   - **Why**: Optimized for time-series, high write throughput
   - **Schema**: Tick data, market metrics

5. **MongoDB** (Document Store)
   - **Use Case**: Wallet snapshots, complex positions
   - **Why**: Flexible schema, nested structures
   - **Schema**: Wallet documents, position documents

**Database Architecture**:
```
Trade Request → Matching Engine
    ├── PostgreSQL → Orders, trades, wallets (ACID)
    ├── Redis → Order book, active orders, prices
    ├── TimescaleDB → Price history, trade history
    ├── Cassandra → Trade history, audit logs
    ├── InfluxDB → Real-time market data, tick data
    └── MongoDB → Wallet snapshots, positions
```

**Selection Matrix**:
| Requirement | PostgreSQL | Redis | TimescaleDB | Cassandra | InfluxDB |
|------------|------------|-------|-------------|-----------|----------|
| Order Matching | ✅ Excellent | ✅ Good | ❌ Poor | ❌ Poor | ❌ Poor |
| Order Book | ⚠️ Moderate | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor |
| Trade History | ✅ Excellent | ❌ Poor | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Price History | ⚠️ Moderate | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ✅ Excellent |
| Real-Time Market Data | ❌ Poor | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ✅ Excellent |

---

### Problem 39: Design a Billing System

#### Requirements
- **Scale**: 10M+ customers, 100M+ invoices/month, complex billing rules
- **Features**: Usage tracking, invoicing, payment processing, reporting
- **Read/Write Ratio**: 10:1 (reads dominate)
- **Data Types**: Usage data, invoices, payments, customers, plans
- **Consistency**: Strong consistency required (financial data)

#### Database Selection Criteria

**Primary Database: PostgreSQL (Relational)**

**Why**:
- **ACID Transactions**: Critical for billing, invoicing
- **Strong Consistency**: Required for financial calculations
- **Complex Queries**: Billing calculations, reporting, analytics
- **Relationships**: Customers, plans, usage, invoices
- **Use Case**: Customers, plans, usage, invoices, payments

**Secondary Databases**:

1. **TimescaleDB** (Time-Series)
   - **Use Case**: Usage metrics, billing metrics, time-based analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Usage over time, billing metrics, cost analysis

2. **Redis** (In-Memory Cache)
   - **Use Case**: Usage counters, rate limits, recent invoices
   - **Why**: Sub-millisecond latency, atomic operations
   - **TTL**: Usage counters (reset period), recent invoices (1 hour)

3. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Usage logs, event logs, audit trail
   - **Why**: High write throughput, time-series data
   - **Schema**: `usage_by_customer`, `events_by_timestamp`

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Invoice search, usage search, analytics
   - **Why**: Full-text search, aggregations, analytics
   - **Schema**: Invoice documents, usage documents

5. **MongoDB** (Document Store)
   - **Use Case**: Invoice details, nested billing data
   - **Why**: Flexible schema, nested structures
   - **Schema**: Invoice documents with line items, details

**Database Architecture**:
```
Usage Event → Billing System
    ├── PostgreSQL → Customers, plans, invoices, payments (ACID)
    ├── TimescaleDB → Usage metrics, billing metrics
    ├── Redis → Usage counters, rate limits
    ├── Cassandra → Usage logs, event logs
    ├── Elasticsearch → Invoice search, analytics
    └── MongoDB → Invoice details, billing data
```

**Selection Matrix**:
| Requirement | PostgreSQL | TimescaleDB | Redis | Cassandra | Elasticsearch |
|------------|------------|-------------|-------|-----------|---------------|
| Billing Calculations | ✅ Excellent | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor |
| Usage Tracking | ⚠️ Moderate | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Invoice Generation | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ⚠️ Moderate |
| Invoice Search | ⚠️ Moderate | ❌ Poor | ❌ Poor | ❌ Poor | ✅ Excellent |
| Usage Analytics | ⚠️ Moderate | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent |

---

### Problem 40: Design a Fraud Detection System

#### Requirements
- **Scale**: 1B+ transactions/day, real-time detection, < 100ms latency
- **Features**: Real-time detection, machine learning, rule engine, alerting
- **Read/Write Ratio**: 1:1 (balanced)
- **Data Types**: Transactions, features, models, rules, alerts
- **Consistency**: Eventual consistency acceptable

#### Database Selection Criteria

**Primary Database: Redis (In-Memory Cache)**

**Why**:
- **Real-Time Detection**: Sub-millisecond feature lookup
- **Feature Store**: ML features, user profiles, risk scores
- **Low Latency**: Required for real-time fraud detection
- **Data Structures**: Hashes for features, sorted sets for risk scores
- **Use Case**: Feature cache, risk scores, real-time rules

**Secondary Databases**:

1. **PostgreSQL** (Relational)
   - **Use Case**: Transaction data, user data, rule configurations
   - **Why**: ACID transactions, complex queries, relationships
   - **Schema**: Transactions, users, rules, configurations

2. **Cassandra** (NoSQL - Wide Column)
   - **Use Case**: Transaction history, feature history, audit logs
   - **Why**: High write throughput, time-series data
   - **Schema**: `transactions_by_user`, `features_by_timestamp`

3. **TimescaleDB** (Time-Series)
   - **Use Case**: Fraud patterns, metrics, analytics
   - **Why**: Time-series optimization, efficient queries
   - **Schema**: Fraud events over time, pattern analysis

4. **Elasticsearch** (Search Engine)
   - **Use Case**: Fraud pattern search, anomaly detection
   - **Why**: Full-text search, aggregations, anomaly detection
   - **Schema**: Fraud events, patterns, anomalies

5. **ML Feature Store** (Specialized)
   - **Use Case**: ML features, model predictions, embeddings
   - **Why**: Feature serving, model predictions
   - **Tools**: Redis, Cassandra, or specialized feature stores

**Database Architecture**:
```
Transaction → Fraud Detection
    ├── Redis → Feature cache, risk scores, real-time rules
    ├── PostgreSQL → Transaction data, user data, rules
    ├── Cassandra → Transaction history, feature history
    ├── TimescaleDB → Fraud patterns, metrics
    ├── Elasticsearch → Fraud pattern search, anomalies
    └── Feature Store → ML features, predictions
```

**Selection Matrix**:
| Requirement | Redis | PostgreSQL | Cassandra | TimescaleDB | Elasticsearch |
|------------|-------|------------|-----------|-------------|---------------|
| Real-Time Detection | ✅ Excellent | ❌ Poor | ❌ Poor | ❌ Poor | ❌ Poor |
| Feature Lookup | ✅ Excellent | ⚠️ Moderate | ⚠️ Moderate | ❌ Poor | ❌ Poor |
| Transaction History | ❌ Poor | ✅ Excellent | ✅ Excellent | ✅ Excellent | ⚠️ Moderate |
| Fraud Pattern Analysis | ❌ Poor | ⚠️ Moderate | ⚠️ Moderate | ✅ Excellent | ✅ Excellent |
| ML Feature Serving | ✅ Excellent | ❌ Poor | ⚠️ Moderate | ❌ Poor | ❌ Poor |

---

## Database Selection Framework (Continued)

### Payment & Financial Patterns

**Pattern 1: Financial Transactions**
- **Requirement**: ACID, strong consistency, audit trail
- **Databases**: PostgreSQL, MySQL, SQL Server
- **Use Cases**: Payments, wallets, exchanges, billing

**Pattern 2: Real-Time Trading**
- **Requirement**: Low latency, high throughput, order book
- **Databases**: Redis (order book), PostgreSQL (trades)
- **Use Cases**: Stock trading, cryptocurrency exchanges

**Pattern 3: Fraud Detection**
- **Requirement**: Real-time, ML features, pattern matching
- **Databases**: Redis (features), PostgreSQL (transactions), ML stores
- **Use Cases**: Payment fraud, transaction fraud

**Pattern 4: Billing & Invoicing**
- **Requirement**: Complex calculations, reporting, analytics
- **Databases**: PostgreSQL (billing), TimescaleDB (usage)
- **Use Cases**: SaaS billing, usage-based billing

---

## Summary

**Problems Covered (36-40)**:
36. Payment Processing System → PostgreSQL, Redis, Cassandra, TimescaleDB, Elasticsearch
37. Wallet System → PostgreSQL, Redis, Cassandra, TimescaleDB, MongoDB
38. Cryptocurrency Exchange → PostgreSQL, Redis, TimescaleDB, Cassandra, InfluxDB
39. Billing System → PostgreSQL, TimescaleDB, Redis, Cassandra, Elasticsearch
40. Fraud Detection System → Redis, PostgreSQL, Cassandra, TimescaleDB, Elasticsearch

**Key Patterns Identified**:
- **Financial Transactions**: PostgreSQL for ACID guarantees
- **Real-Time Trading**: Redis for order book, PostgreSQL for trades
- **Fraud Detection**: Redis for real-time features, PostgreSQL for transactions
- **Billing**: PostgreSQL for calculations, TimescaleDB for usage metrics
- **Audit Trail**: PostgreSQL or Cassandra for transaction history

**Next**: Problems 41-45 (Infrastructure & DevOps Systems)

