# System Design Interview Tips - Part 1: Clarify Requirements & Start High-Level

## Overview

This document provides in-depth guidance on the first two critical tips for system design interviews:
1. **Clarify Requirements**: Ask about scale, constraints, and priorities
2. **Start High-Level**: Draw architecture diagram first

---

## 1. Clarify Requirements: Ask About Scale, Constraints, and Priorities

### 1.1 Why Clarifying Requirements is Critical

**Common Mistakes**:
- Jumping into design without understanding requirements
- Making assumptions about scale or constraints
- Designing for wrong use case
- Over-engineering or under-engineering

**Benefits of Clarifying**:
- Design the right solution for the problem
- Avoid unnecessary complexity
- Focus on what matters most
- Demonstrate analytical thinking

### 1.2 What to Ask: The Requirement Gathering Framework

#### A. Functional Requirements

**Questions to Ask**:
1. **Core Features**: What are the must-have features?
2. **User Actions**: What can users do?
3. **Data Model**: What data needs to be stored?
4. **Workflows**: What are the main user flows?

**Example: Design a Chat Application**

```
❌ Bad Approach:
"I'll design a chat app with WebSocket, Redis, and PostgreSQL."

✅ Good Approach:
"Let me clarify the requirements:
- Is this 1-on-1 chat or group chat or both?
- Do we need message history?
- Do we need read receipts?
- Do we need file sharing?
- Do we need voice/video calls?
- What's the message size limit?"
```

**Clarified Requirements**:
- 1-on-1 and group chats (up to 100 members)
- Message history for 30 days
- Read receipts required
- File sharing up to 10MB
- No voice/video calls initially
- Message size: 1KB average, 10KB max

#### B. Scale Requirements

**Questions to Ask**:
1. **Users**: How many users? Daily active users (DAU)? Monthly active users (MAU)?
2. **Traffic**: Requests per second (RPS)? Peak traffic?
3. **Data Volume**: Data size? Growth rate?
4. **Geographic Distribution**: Global? Regional?

**Example: Design a URL Shortener**

```
Questions to Ask:
1. "How many URLs do we need to shorten per day?"
   → Answer: 100 million URLs/day

2. "What's the read-to-write ratio?"
   → Answer: 100:1 (100 reads per write)

3. "What's the average URL size?"
   → Answer: 500 bytes

4. "How long should shortened URLs be valid?"
   → Answer: Indefinitely (or 5 years)

5. "What's the peak traffic?"
   → Answer: 10x average (1 billion reads/day peak)
```

**Calculations**:
```
Writes: 100M URLs/day = 100M / (24 * 3600) = ~1,160 URLs/sec
Reads: 100 * 1,160 = 116,000 reads/sec
Peak Reads: 1,160,000 reads/sec

Storage: 100M URLs/day * 500 bytes = 50GB/day
         = 1.5TB/month = 18TB/year
```

#### C. Constraints and Priorities

**Questions to Ask**:
1. **Latency**: What's acceptable latency? P50, P99?
2. **Consistency**: Strong consistency or eventual consistency?
3. **Availability**: What's the uptime requirement? (e.g., 99.9%, 99.99%)
4. **Durability**: Can we lose data? What's the RPO (Recovery Point Objective)?
5. **Cost**: Any budget constraints?
6. **Time to Market**: When does this need to be ready?

**Example: Design a Payment System**

```
Questions to Ask:
1. "What's the acceptable latency for payment processing?"
   → Answer: P99 < 500ms

2. "Do we need strong consistency for balance updates?"
   → Answer: Yes, absolutely (ACID required)

3. "What's the availability requirement?"
   → Answer: 99.99% (52 minutes downtime/year)

4. "Can we lose any payment transactions?"
   → Answer: No, zero data loss tolerance

5. "What's the budget for infrastructure?"
   → Answer: Cost-optimized, but reliability first
```

#### D. Non-Functional Requirements

**Questions to Ask**:
1. **Security**: Authentication? Authorization? Encryption?
2. **Compliance**: GDPR? PCI-DSS? HIPAA?
3. **Monitoring**: What metrics are important?
4. **Scalability**: Expected growth? 10x? 100x?
5. **Maintainability**: Team size? Tech stack preferences?

**Example: Design a Healthcare System**

```
Questions to Ask:
1. "What compliance requirements do we need?"
   → Answer: HIPAA compliance required

2. "What authentication mechanism?"
   → Answer: OAuth 2.0 with MFA

3. "What's the data retention policy?"
   → Answer: 7 years (legal requirement)

4. "What's the team's tech stack preference?"
   → Answer: Java/Spring Boot, PostgreSQL
```

### 1.3 Requirement Clarification Template

**Use this template during interviews**:

```
1. FUNCTIONAL REQUIREMENTS
   - Core features: [list]
   - User actions: [list]
   - Data model: [describe]
   - Workflows: [describe]

2. SCALE REQUIREMENTS
   - Users: [DAU, MAU]
   - Traffic: [RPS, peak RPS]
   - Data: [size, growth rate]
   - Geography: [global/regional]

3. CONSTRAINTS
   - Latency: [P50, P99]
   - Consistency: [strong/eventual]
   - Availability: [99.9%, 99.99%]
   - Durability: [RPO, RTO]
   - Cost: [budget constraints]

4. PRIORITIES
   - What's most important? [latency/consistency/availability]
   - What can we optimize later? [list]
   - What's out of scope? [list]
```

### 1.4 Common Pitfalls to Avoid

**Pitfall 1: Assuming Scale**
```
❌ "I'll design for 1 million users"
✅ "What's the expected user base? 1M? 10M? 100M?"
```

**Pitfall 2: Ignoring Constraints**
```
❌ Designing eventual consistency for payment system
✅ "Do we need strong consistency for financial transactions?"
```

**Pitfall 3: Not Prioritizing**
```
❌ Trying to optimize everything
✅ "What's the most critical requirement? Latency or consistency?"
```

**Pitfall 4: Scope Creep**
```
❌ Adding features not asked for
✅ "Should I include [feature] or focus on core functionality first?"
```

### 1.5 Example: Complete Requirement Clarification

**Problem**: Design a News Feed System

```
INTERVIEWER: "Design a news feed system like Facebook."

YOU: "Let me clarify a few requirements:

1. Functional:
   - Is this for a single user's feed or can users follow others?
   - Do we need to support posts, photos, videos, or all?
   - Do we need comments, likes, shares?
   - Do we need real-time updates or is eventual consistency okay?

2. Scale:
   - How many users? Daily active users?
   - How many posts per user per day?
   - What's the read-to-write ratio?
   - What's the peak traffic?

3. Constraints:
   - What's acceptable feed generation latency?
   - Do we need to show posts in chronological order or ranked?
   - What's the availability requirement?

4. Priorities:
   - What's more important: freshness or relevance?
   - Should I focus on read path or write path optimization?"
```

**Clarified Requirements**:
```
Functional:
- Users can follow other users
- Posts, photos, videos supported
- Comments, likes, shares included
- Real-time updates preferred

Scale:
- 1 billion users, 500M DAU
- 10 posts/user/day average
- 100:1 read-to-write ratio
- Peak: 10x average

Constraints:
- Feed generation: P99 < 200ms
- Ranked feed (not chronological)
- 99.9% availability

Priorities:
- Relevance > Freshness
- Read path optimization critical
```

---

## 2. Start High-Level: Draw Architecture Diagram First

### 2.1 Why Start High-Level

**Benefits**:
- Shows big-picture thinking
- Identifies major components early
- Helps identify bottlenecks
- Guides detailed design
- Demonstrates communication skills

**Common Mistakes**:
- Jumping into database schema details
- Focusing on implementation details too early
- Missing major components
- Not showing data flow

### 2.2 High-Level Architecture Components

#### A. Standard Components

```
┌─────────────────────────────────────────────────────────────┐
│                    High-Level Architecture                   │
└─────────────────────────────────────────────────────────────┘

1. CLIENT LAYER
   - Web clients
   - Mobile apps
   - Desktop apps
   - Third-party integrations

2. EDGE LAYER
   - CDN
   - Load Balancer
   - API Gateway

3. APPLICATION LAYER
   - Web servers
   - Application servers
   - Microservices

4. DATA LAYER
   - Databases (SQL/NoSQL)
   - Caches
   - Message queues
   - Object storage

5. INFRASTRUCTURE
   - Monitoring
   - Logging
   - Security
```

#### B. Drawing the Architecture

**Step 1: Identify Main Components**

```
For a Chat Application:

Components:
- Client (Web, Mobile)
- API Gateway
- Chat Service
- Message Queue
- Database
- Cache
- WebSocket Service
```

**Step 2: Draw Data Flow**

```
User sends message:
Client → API Gateway → Chat Service → Message Queue → Database
                                    ↓
                              WebSocket Service → Other Client
```

**Step 3: Add Supporting Services**

```
- Authentication Service
- Notification Service
- File Storage Service
- Analytics Service
```

### 2.3 High-Level Architecture Template

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENT LAYER                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                 │
│  │   Web    │  │  Mobile  │  │  Desktop │                 │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                 │
└───────┼──────────────┼──────────────┼───────────────────────┘
        │              │              │
        └──────────────┼──────────────┘
                       │
        ┌──────────────▼──────────────┐
        │      API GATEWAY /          │
        │      LOAD BALANCER           │
        └──────────────┬──────────────┘
                       │
        ┌──────────────▼──────────────┐
        │    APPLICATION SERVICES     │
        │  ┌────────┐  ┌────────┐    │
        │  │Service1│  │Service2│    │
        │  └───┬────┘  └───┬────┘    │
        └──────┼───────────┼──────────┘
               │           │
        ┌──────▼───────────▼──────┐
        │      DATA LAYER         │
        │  ┌──────┐  ┌──────┐    │
        │  │  DB  │  │Cache │    │
        │  └──────┘  └──────┘    │
        └─────────────────────────┘
```

### 2.4 Example: High-Level Architecture for URL Shortener

```
┌─────────────────────────────────────────────────────────────┐
│                    URL SHORTENER SYSTEM                      │
└─────────────────────────────────────────────────────────────┘

┌──────────────┐
│   Clients    │
│ (Web/Mobile) │
└──────┬───────┘
       │
       │ HTTPS
       │
┌──────▼──────────────────────────────────────┐
│         API GATEWAY / LOAD BALANCER         │
│  - Request routing                           │
│  - Rate limiting                             │
│  - SSL termination                           │
└──────┬──────────────────────────────────────┘
       │
       ├─────────────────┬─────────────────┐
       │                 │                 │
┌──────▼──────┐   ┌──────▼──────┐   ┌──────▼──────┐
│  Short URL  │   │  Redirect   │   │  Analytics  │
│   Service   │   │   Service    │   │   Service   │
└──────┬──────┘   └──────┬──────┘   └──────┬──────┘
       │                 │                 │
       │                 │                 │
┌──────▼─────────────────▼─────────────────▼──────┐
│              DATA LAYER                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │Database  │  │  Cache   │  │ Analytics│       │
│  │(URLs)     │  │(Hot URLs)│  │   DB     │       │
│  └──────────┘  └──────────┘  └──────────┘       │
└──────────────────────────────────────────────────┘
```

### 2.5 Best Practices for High-Level Design

#### Practice 1: Use Standard Symbols

```
┌─────┐  =  Service/Component
  │      =  Data Flow
  ▼      =  Direction
┌───┐   =  Database
│   │   =  Queue
```

#### Practice 2: Show Data Flow

```
Always show:
1. Request flow (client → server)
2. Response flow (server → client)
3. Async flows (queues, events)
4. Data replication flows
```

#### Practice 3: Label Everything

```
Don't use generic names:
❌ "Service", "Database", "Cache"

Use specific names:
✅ "URL Shortening Service", "PostgreSQL", "Redis"
```

#### Practice 4: Show Scale Indicators

```
Indicate if components are:
- Single instance
- Multiple instances (with "x3", "x5")
- Distributed/clustered
```

### 2.6 Common High-Level Design Patterns

#### Pattern 1: Three-Tier Architecture

```
Client → Application Server → Database
```

#### Pattern 2: Microservices Architecture

```
Client → API Gateway → [Service1, Service2, Service3] → Databases
```

#### Pattern 3: Event-Driven Architecture

```
Client → Service → Message Queue → [Consumer1, Consumer2] → Databases
```

#### Pattern 4: CQRS (Command Query Responsibility Segregation)

```
Write: Client → Command Service → Write DB
Read:  Client → Query Service → Read DB (replicated)
```

### 2.7 Example: Complete High-Level Design Process

**Problem**: Design a News Feed System

**Step 1: Identify Components**

```
Components needed:
- Client (Web/Mobile)
- API Gateway
- Feed Service (generates feeds)
- Post Service (creates posts)
- User Service (user data)
- Social Graph Service (followers)
- Ranking Service (relevance)
- Cache (feed cache)
- Database (posts, users, graph)
- Message Queue (async updates)
```

**Step 2: Draw Architecture**

```
┌─────────────────────────────────────────────────────────────┐
│                    NEWS FEED SYSTEM                         │
└─────────────────────────────────────────────────────────────┘

┌──────────────┐
│   Clients    │
└──────┬───────┘
       │
┌──────▼──────────────────────────────────────┐
│         API GATEWAY                         │
└──────┬──────────────────────────────────────┘
       │
       ├──────────────┬──────────────┬──────────────┐
       │              │              │              │
┌──────▼──────┐  ┌───▼──────┐  ┌───▼──────┐  ┌───▼──────┐
│   Feed      │  │   Post    │  │   User   │  │  Social  │
│  Service    │  │  Service  │  │ Service │  │  Graph   │
└──────┬──────┘  └───┬──────┘  └───┬──────┘  └───┬──────┘
       │              │              │              │
       │              │              │              │
       └──────────────┼──────────────┼──────────────┘
                      │              │
              ┌───────▼──────────────▼───────┐
              │      RANKING SERVICE          │
              └───────┬───────────────────────┘
                      │
       ┌──────────────┼──────────────┐
       │              │              │
┌──────▼──────┐  ┌───▼──────┐  ┌───▼──────┐
│  Database   │  │   Cache  │  │  Queue   │
│ (Posts/User)│  │ (Feeds)   │  │ (Events) │
└─────────────┘  └───────────┘  └──────────┘
```

**Step 3: Show Data Flow**

```
Feed Request Flow:
Client → API Gateway → Feed Service → Cache (check)
                                    ↓ (miss)
                              Social Graph Service → Get followers
                              Post Service → Get posts
                              Ranking Service → Rank posts
                              Cache (store) → Return feed

Post Creation Flow:
Client → API Gateway → Post Service → Database (store)
                                    → Queue (publish event)
                                    → Feed Service (invalidate cache)
```

### 2.8 Interview Tips

**Tip 1: Draw as You Talk**
- Don't wait to draw everything
- Draw and explain simultaneously
- Iterate based on feedback

**Tip 2: Ask for Feedback**
- "Does this architecture make sense?"
- "Should I add/remove any components?"
- "Is this the right level of detail?"

**Tip 3: Be Flexible**
- Be ready to modify based on feedback
- Don't be attached to initial design
- Show adaptability

**Tip 4: Explain Your Choices**
- Why this component?
- Why this pattern?
- What are the alternatives?

---

## Summary

### Clarify Requirements
1. ✅ Ask about functional requirements
2. ✅ Understand scale (users, traffic, data)
3. ✅ Identify constraints (latency, consistency, availability)
4. ✅ Determine priorities
5. ✅ Use a structured template

### Start High-Level
1. ✅ Draw architecture diagram first
2. ✅ Identify main components
3. ✅ Show data flow
4. ✅ Use standard patterns
5. ✅ Iterate based on feedback

**Key Takeaway**: Spending 5-10 minutes clarifying requirements and drawing high-level architecture will save time and lead to a better design.

