# URL Shortener System Design (like bit.ly)

## Requirements

### Functional Requirements
- Shorten long URLs to short aliases (e.g., `bit.ly/abc123`)
- Redirect short URLs to original URLs
- Support custom aliases (optional)
- URL expiration (optional TTL)
- Analytics: track click counts, referrers, geographic data

### Non-Functional Requirements
- **Scale**: 100M URLs/month, 10B redirects/month
- **Latency**: Redirect should be < 100ms (P95)
- **Availability**: 99.9% uptime
- **Durability**: No data loss
- **Security**: Prevent abuse, malicious URLs

## Capacity Estimation

### Traffic Estimates
- **URL Creation**: 100M/month = ~38 URLs/sec (peak ~1000/sec with 30x spike)
- **Redirects**: 10B/month = ~3,858 redirects/sec (peak ~115K/sec)
- **Read:Write Ratio**: ~100:1 (heavily read-heavy)

### Storage Estimates
- **Short URL**: 6-8 characters = ~8 bytes
- **Long URL**: Average 100 characters = ~100 bytes
- **Metadata**: Created timestamp, expiration, user_id, click_count = ~50 bytes
- **Total per record**: ~158 bytes
- **100M URLs**: 100M × 158 bytes = ~15.8 GB
- **With 5-year retention + 10x growth**: ~800 GB

### Bandwidth Estimates
- **Writes**: 100 URLs/sec × 158 bytes = ~15.8 KB/sec
- **Reads**: 3,858 redirects/sec × 158 bytes = ~610 KB/sec
- **Total**: ~625 KB/sec = ~5 Mbps

## System Architecture

### High-Level Design

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
       ▼
┌─────────────────────────────────────┐
│         Load Balancer               │
└──────┬──────────────────┬────────────┘
       │                  │
       ▼                  ▼
┌─────────────┐    ┌─────────────┐
│  API Server │    │  API Server │
│  (Writes)   │    │  (Reads)    │
└──────┬──────┘    └──────┬──────┘
       │                  │
       │                  ▼
       │          ┌─────────────┐
       │          │   Redis     │
       │          │   Cache     │
       │          └──────┬──────┘
       │                 │
       ▼                 ▼
┌─────────────────────────────────┐
│      Database (Sharded)          │
│  ┌──────┐  ┌──────┐  ┌──────┐   │
│  │Shard1│  │Shard2│  │ShardN│   │
│  └──────┘  └──────┘  └──────┘   │
└─────────────────────────────────┘
```

## Core Components

### 1. Hash Function for Short URLs

#### Option A: Base62 Encoding of Auto-Increment ID
- **Pros**: Simple, sequential, no collisions
- **Cons**: Predictable, exposes growth rate, needs distributed ID generator
- **Implementation**: 
  - Use distributed ID generator (Snowflake, UUID, or database sequence per shard)
  - Encode to Base62: `[0-9a-zA-Z]` = 62 characters
  - 6 characters = 62^6 = ~56.8 billion unique URLs
  - 7 characters = 62^7 = ~3.5 trillion unique URLs

#### Option B: Hash-Based (MD5/SHA-256)
- **Pros**: Deterministic, no coordination needed
- **Cons**: Collisions possible, longer hash needed
- **Implementation**:
  - Hash long URL → take first 6-8 characters
  - Handle collisions: append counter or use longer hash
  - Example: `MD5(url)[:6]` or `SHA256(url)[:8]`

#### Option C: Hybrid Approach (Recommended)
- **For auto-generated**: Use Base62 encoding of distributed ID
- **For custom aliases**: Validate uniqueness, store directly
- **Collision handling**: Check existence before assignment

**Recommended: Base62 of Distributed ID**
```java
public class ShortUrlGenerator {
    private static final String BASE62 = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    
    public String encode(long id) {
        StringBuilder sb = new StringBuilder();
        while (id > 0) {
            sb.append(BASE62.charAt((int)(id % 62)));
            id /= 62;
        }
        return sb.reverse().toString();
    }
    
    public long decode(String shortUrl) {
        long id = 0;
        for (char c : shortUrl.toCharArray()) {
            id = id * 62 + BASE62.indexOf(c);
        }
        return id;
    }
}
```

### 2. Database Sharding Strategy

#### Sharding Key: Short URL Hash
- **Approach**: Hash the short URL to determine shard
- **Shard Count**: Start with 10 shards, scale to 100+ as needed
- **Shard Selection**: `shard_id = hash(short_url) % num_shards`

#### Alternative: Range-Based Sharding
- **Approach**: Partition by ID ranges
- **Pros**: Easier range queries, simpler rebalancing
- **Cons**: Potential hotspots if IDs are sequential

#### Database Schema (Per Shard)

```sql
CREATE TABLE short_urls (
    id BIGINT PRIMARY KEY,
    short_code VARCHAR(8) UNIQUE NOT NULL,
    long_url TEXT NOT NULL,
    user_id VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    expires_at TIMESTAMP,
    click_count BIGINT DEFAULT 0,
    INDEX idx_short_code (short_code),
    INDEX idx_user_id (user_id),
    INDEX idx_expires_at (expires_at)
);

CREATE TABLE url_analytics (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    short_code VARCHAR(8) NOT NULL,
    clicked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    referrer TEXT,
    country VARCHAR(2),
    INDEX idx_short_code_time (short_code, clicked_at)
);
```

#### Shard Routing Logic

```java
@Service
public class ShardRouter {
    private final int numShards = 10;
    
    public int getShard(String shortCode) {
        return Math.abs(shortCode.hashCode()) % numShards;
    }
    
    public String getShardConnectionString(String shortCode) {
        int shardId = getShard(shortCode);
        return "jdbc:postgresql://shard-" + shardId + ".db:5432/urlshortener";
    }
}
```

### 3. Caching Strategy

#### Multi-Layer Caching

**Layer 1: Application Cache (In-Memory)**
- **Technology**: Caffeine/Guava Cache
- **Size**: ~10K most popular URLs
- **TTL**: 1 hour
- **Eviction**: LRU

**Layer 2: Distributed Cache (Redis)**
- **Technology**: Redis Cluster
- **Size**: ~100M URLs (hot set)
- **TTL**: 24 hours (configurable)
- **Pattern**: Cache-aside
- **Key Format**: `url:{short_code}`
- **Value**: JSON with `{long_url, expires_at, click_count}`

#### Cache Strategy: Cache-Aside

```java
@Service
public class UrlService {
    private final RedisTemplate<String, String> redis;
    private final ShardRouter shardRouter;
    
    public String getLongUrl(String shortCode) {
        // 1. Check Redis cache
        String cached = redis.opsForValue().get("url:" + shortCode);
        if (cached != null) {
            return parseLongUrl(cached);
        }
        
        // 2. Check database (routed to correct shard)
        int shardId = shardRouter.getShard(shortCode);
        String longUrl = databaseService.getLongUrl(shardId, shortCode);
        
        if (longUrl != null) {
            // 3. Populate cache
            redis.opsForValue().set("url:" + shortCode, longUrl, 
                Duration.ofHours(24));
        }
        
        return longUrl;
    }
}
```

#### Cache Warming
- Pre-load popular URLs on startup
- Background job to refresh cache for high-traffic URLs
- Use Redis with replication for high availability

## API Design

### REST Endpoints

```
POST /api/v1/urls
Request: {
  "long_url": "https://example.com/very/long/url",
  "custom_alias": "my-link" (optional),
  "expires_in_days": 30 (optional)
}
Response: {
  "short_url": "https://short.ly/abc123",
  "expires_at": "2024-02-01T00:00:00Z"
}

GET /{short_code}
Response: 302 Redirect to long_url
Headers: Location: <long_url>

GET /api/v1/urls/{short_code}/stats
Response: {
  "short_url": "https://short.ly/abc123",
  "long_url": "https://example.com/...",
  "click_count": 12345,
  "created_at": "2024-01-01T00:00:00Z"
}
```

## Detailed Component Design

### URL Shortening Service

```java
@Service
public class UrlShorteningService {
    private final DistributedIdGenerator idGenerator;
    private final ShortUrlGenerator urlGenerator;
    private final ShardRouter shardRouter;
    private final RedisTemplate<String, String> redis;
    
    public ShortUrlResponse shortenUrl(CreateUrlRequest request) {
        // 1. Validate URL
        validateUrl(request.getLongUrl());
        
        // 2. Check for custom alias
        String shortCode;
        if (request.getCustomAlias() != null) {
            shortCode = request.getCustomAlias();
            if (isShortCodeExists(shortCode)) {
                throw new ShortCodeAlreadyExistsException();
            }
        } else {
            // 3. Generate unique short code
            long id = idGenerator.nextId();
            shortCode = urlGenerator.encode(id);
        }
        
        // 4. Determine shard
        int shardId = shardRouter.getShard(shortCode);
        
        // 5. Store in database
        UrlEntity entity = new UrlEntity();
        entity.setShortCode(shortCode);
        entity.setLongUrl(request.getLongUrl());
        entity.setExpiresAt(calculateExpiration(request.getExpiresInDays()));
        databaseService.save(shardId, entity);
        
        // 6. Cache for immediate availability
        cacheUrl(shortCode, request.getLongUrl());
        
        return new ShortUrlResponse(shortCode, request.getLongUrl());
    }
}
```

### Redirect Service

```java
@Service
public class RedirectService {
    private final UrlService urlService;
    private final AnalyticsService analyticsService;
    
    public ResponseEntity<Void> redirect(String shortCode, HttpServletRequest request) {
        // 1. Get long URL (with caching)
        String longUrl = urlService.getLongUrl(shortCode);
        
        if (longUrl == null) {
            return ResponseEntity.notFound().build();
        }
        
        // 2. Check expiration
        if (isExpired(shortCode)) {
            return ResponseEntity.status(410).build(); // Gone
        }
        
        // 3. Track analytics (async)
        analyticsService.trackClick(shortCode, request);
        
        // 4. Increment click count (async)
        urlService.incrementClickCount(shortCode);
        
        // 5. Redirect
        return ResponseEntity.status(302)
            .header("Location", longUrl)
            .build();
    }
}
```

## Scaling Strategies

### Horizontal Scaling
- **API Servers**: Stateless, scale horizontally with load balancer
- **Database Shards**: Add shards and rebalance data
- **Redis Cluster**: Scale horizontally with consistent hashing

### Read Replicas
- **Database**: Read replicas per shard for read-heavy workload
- **Routing**: Route reads to replicas, writes to primary

### CDN Integration
- Cache redirects at CDN edge (CloudFront, Cloudflare)
- TTL: 1-24 hours depending on URL volatility
- Reduces database/Redis load significantly

## Data Consistency

### Write Path
1. Write to database (primary shard)
2. Invalidate cache (or write-through)
3. Return success

### Read Path
1. Check cache (Redis)
2. If miss, read from database (replica if available)
3. Populate cache
4. Return result

### Eventual Consistency
- Cache may be slightly stale (acceptable for redirects)
- Click counts updated asynchronously
- Analytics processed in background

## Security Considerations

### URL Validation
- **Malicious URLs**: Check against blacklist, validate protocol (http/https only)
- **Phishing**: Rate limit per IP, CAPTCHA for suspicious patterns
- **Spam**: Rate limit URL creation per user/IP

### Short Code Security
- **Randomization**: Use cryptographically secure random for IDs
- **Collision Prevention**: Check existence before assignment
- **Custom Aliases**: Validate format, prevent reserved words

### Rate Limiting
- **Per IP**: 100 URLs/hour, 1000 redirects/minute
- **Per User**: 10,000 URLs/day (if authenticated)
- **Implementation**: Redis-based sliding window

## Monitoring & Observability

### Key Metrics
- **Latency**: P50, P95, P99 for redirects
- **Throughput**: URLs created/sec, redirects/sec
- **Cache Hit Rate**: Target > 90%
- **Error Rate**: 4xx, 5xx responses
- **Database Connection Pool**: Active connections per shard

### Alerts
- Redirect latency > 200ms (P95)
- Cache hit rate < 80%
- Database connection pool exhaustion
- Shard imbalance > 20%

## Disaster Recovery

### Backup Strategy
- **Database**: Daily full backups, hourly incremental
- **Redis**: Snapshot every 6 hours
- **Retention**: 30 days

### Failover
- **Database**: Automated failover to replica (RTO < 5 min)
- **Redis**: Cluster mode with automatic failover
- **Multi-Region**: Active-passive setup for DR

## Cost Optimization

### Storage
- **Database**: Use compression for long URLs
- **Cache**: TTL-based eviction, only cache hot URLs
- **Analytics**: Move old data to cold storage (S3)

### Compute
- **Auto-scaling**: Scale down during low traffic
- **Reserved Instances**: For predictable baseline load
- **Spot Instances**: For non-critical workloads

## Trade-offs

### Consistency vs. Availability
- **Choice**: Eventual consistency (AP)
- **Rationale**: Redirects can tolerate slight cache staleness
- **Trade-off**: Acceptable for high availability

### Cache vs. Database
- **Choice**: Aggressive caching (90%+ hit rate target)
- **Rationale**: 100:1 read:write ratio
- **Trade-off**: Slightly stale data acceptable

### Sharding Complexity
- **Choice**: Hash-based sharding
- **Rationale**: Even distribution, simple routing
- **Trade-off**: Rebalancing requires data movement

## Future Enhancements

1. **Custom Domains**: Allow users to use their own domain
2. **QR Codes**: Generate QR codes for short URLs
3. **Bulk Operations**: Batch URL creation
4. **Advanced Analytics**: Geographic heatmaps, device breakdown
5. **Link Preview**: Generate preview cards for shared links
6. **Password Protection**: Require password for sensitive URLs

## Implementation Checklist

- [ ] Distributed ID generator (Snowflake/UUID)
- [ ] Base62 encoding/decoding
- [ ] Database sharding with routing logic
- [ ] Redis cache with cache-aside pattern
- [ ] URL validation and security checks
- [ ] Rate limiting (Redis-based)
- [ ] Analytics tracking (async)
- [ ] Monitoring and alerting
- [ ] Load testing (target: 115K redirects/sec)
- [ ] Disaster recovery procedures

