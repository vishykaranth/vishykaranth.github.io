# System Architecture for Scale: In-Depth Guide

## Part 2: Caching Strategies and Database Scaling

---

## Table of Contents

1. [Multi-Layer Caching](#1-multi-layer-caching)
2. [Cache Patterns](#2-cache-patterns)
3. [Database Scaling Techniques](#3-database-scaling-techniques)
4. [Sharding Strategies](#4-sharding-strategies)
5. [Practical Examples](#5-practical-examples)

---

## 1. Multi-Layer Caching

### 1.1 Caching Layers

**Caching Hierarchy**:
```
Client
  │
  ▼
Browser Cache (L1)
  │
  ▼
CDN Cache (L2)
  │
  ▼
Application Cache (L3) - Redis/Memcached
  │
  ▼
Database Cache (L4) - Query cache, Buffer pool
  │
  ▼
Database
```

### 1.2 Browser Cache Implementation

```java
@RestController
@RequestMapping("/api/products")
class ProductController {
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable String id) {
        Product product = productService.getProduct(id);
        
        // Set cache headers
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(1, TimeUnit.HOURS))
            .eTag(product.getVersion().toString())
            .lastModified(product.getLastModified())
            .body(product);
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<Product> getProductWithETag(
            @PathVariable String id,
            @RequestHeader(value = "If-None-Match", required = false) String ifNoneMatch) {
        
        Product product = productService.getProduct(id);
        String etag = product.getVersion().toString();
        
        // Return 304 Not Modified if ETag matches
        if (etag.equals(ifNoneMatch)) {
            return ResponseEntity.status(HttpStatus.NOT_MODIFIED)
                .eTag(etag)
                .build();
        }
        
        return ResponseEntity.ok()
            .eTag(etag)
            .body(product);
    }
}
```

### 1.3 Application-Level Caching

**Redis Cache Implementation**:
```java
@Configuration
class CacheConfig {
    
    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        return new LettuceConnectionFactory(
            new RedisStandaloneConfiguration("localhost", 6379));
    }
    
    @Bean
    public RedisTemplate<String, Object> redisTemplate() {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(redisConnectionFactory());
        template.setDefaultSerializer(new GenericJackson2JsonRedisSerializer());
        return template;
    }
    
    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            .serializeValuesWith(RedisSerializationContext.SerializationPair
                .fromSerializer(new GenericJackson2JsonRedisSerializer()));
        
        return RedisCacheManager.builder(factory)
            .cacheDefaults(config)
            .build();
    }
}

// Service with caching
@Service
class ProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private RedisTemplate<String, Product> redisTemplate;
    
    private static final String CACHE_KEY_PREFIX = "product:";
    private static final String CACHE_LIST_KEY = "products:list";
    
    @Cacheable(value = "products", key = "#id")
    public Product getProduct(String id) {
        return productRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("Product not found"));
    }
    
    @CacheEvict(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        Product updated = productRepository.save(product);
        
        // Invalidate related caches
        redisTemplate.delete(CACHE_LIST_KEY);
        
        return updated;
    }
    
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(String id) {
        productRepository.deleteById(id);
        redisTemplate.delete(CACHE_LIST_KEY);
    }
    
    // Cache-aside pattern
    public List<Product> getAllProducts() {
        // Check cache first
        List<Product> cached = (List<Product>) redisTemplate.opsForValue()
            .get(CACHE_LIST_KEY);
        
        if (cached != null) {
            return cached;
        }
        
        // Cache miss - fetch from database
        List<Product> products = productRepository.findAll();
        
        // Store in cache
        redisTemplate.opsForValue().set(CACHE_LIST_KEY, products, 
            Duration.ofMinutes(30));
        
        return products;
    }
}
```

### 1.4 Cache-Aside Pattern

```java
@Service
class CacheAsideService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private RedisTemplate<String, User> redisTemplate;
    
    private static final String CACHE_KEY_PREFIX = "user:";
    private static final Duration CACHE_TTL = Duration.ofHours(1);
    
    public User getUser(String id) {
        // 1. Check cache
        String cacheKey = CACHE_KEY_PREFIX + id;
        User cached = redisTemplate.opsForValue().get(cacheKey);
        
        if (cached != null) {
            return cached; // Cache hit
        }
        
        // 2. Cache miss - read from database
        User user = userRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("User not found"));
        
        // 3. Write to cache
        redisTemplate.opsForValue().set(cacheKey, user, CACHE_TTL);
        
        return user;
    }
    
    public User updateUser(User user) {
        // 1. Update database
        User updated = userRepository.save(user);
        
        // 2. Update cache
        String cacheKey = CACHE_KEY_PREFIX + user.getId();
        redisTemplate.opsForValue().set(cacheKey, updated, CACHE_TTL);
        
        return updated;
    }
    
    public void deleteUser(String id) {
        // 1. Delete from database
        userRepository.deleteById(id);
        
        // 2. Delete from cache
        String cacheKey = CACHE_KEY_PREFIX + id;
        redisTemplate.delete(cacheKey);
    }
}
```

### 1.5 Write-Through Pattern

```java
@Service
class WriteThroughService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private RedisTemplate<String, Product> redisTemplate;
    
    public Product createProduct(Product product) {
        // 1. Write to cache first
        String cacheKey = "product:" + product.getId();
        redisTemplate.opsForValue().set(cacheKey, product, Duration.ofHours(1));
        
        // 2. Write to database
        return productRepository.save(product);
    }
    
    public Product updateProduct(Product product) {
        // 1. Update cache
        String cacheKey = "product:" + product.getId();
        redisTemplate.opsForValue().set(cacheKey, product, Duration.ofHours(1));
        
        // 2. Update database
        return productRepository.save(product);
    }
}
```

### 1.6 Write-Behind Pattern

```java
@Service
class WriteBehindService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private RedisTemplate<String, Order> redisTemplate;
    
    private BlockingQueue<Order> writeQueue = new LinkedBlockingQueue<>();
    
    @PostConstruct
    public void init() {
        // Background thread for batch writes
        new Thread(this::batchWriteToDatabase).start();
    }
    
    public Order createOrder(Order order) {
        // 1. Write to cache immediately
        String cacheKey = "order:" + order.getId();
        redisTemplate.opsForValue().set(cacheKey, order, Duration.ofDays(7));
        
        // 2. Queue for database write
        writeQueue.offer(order);
        
        return order;
    }
    
    private void batchWriteToDatabase() {
        List<Order> batch = new ArrayList<>();
        
        while (true) {
            try {
                // Collect orders for batch write
                Order order = writeQueue.poll(5, TimeUnit.SECONDS);
                if (order != null) {
                    batch.add(order);
                }
                
                // Batch write when size reaches threshold or timeout
                if (batch.size() >= 100 || (order == null && !batch.isEmpty())) {
                    orderRepository.saveAll(batch);
                    batch.clear();
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    }
}
```

---

## 2. Cache Patterns

### 2.1 Cache-Aside (Lazy Loading)

**Flow**:
```
1. Application checks cache
2. If cache hit → return data
3. If cache miss → read from database
4. Write to cache
5. Return data
```

**Implementation**:
```java
@Service
class CacheAsideService {
    
    public User getUser(String id) {
        // Check cache
        User cached = cache.get("user:" + id);
        if (cached != null) {
            return cached;
        }
        
        // Read from database
        User user = repository.findById(id);
        
        // Write to cache
        cache.put("user:" + id, user);
        
        return user;
    }
}
```

### 2.2 Read-Through Pattern

**Flow**:
```
1. Application requests data
2. Cache automatically loads from database if miss
3. Returns data to application
```

**Implementation**:
```java
@Service
class ReadThroughService {
    
    @Cacheable(value = "users", key = "#id")
    public User getUser(String id) {
        // Spring Cache automatically handles read-through
        return repository.findById(id);
    }
}
```

### 2.3 Cache Invalidation Strategies

**Time-Based Expiration**:
```java
@Service
class TimeBasedCacheService {
    
    @Cacheable(value = "products", 
               key = "#id",
               unless = "#result == null")
    public Product getProduct(String id) {
        return repository.findById(id);
    }
    
    // Cache expires after 1 hour (configured in CacheConfig)
}
```

**Event-Based Invalidation**:
```java
@Component
class CacheInvalidationListener {
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;
    
    @EventListener
    public void handleProductUpdated(ProductUpdatedEvent event) {
        // Invalidate specific cache
        redisTemplate.delete("product:" + event.getProductId());
        
        // Invalidate related caches
        redisTemplate.delete("products:list");
        redisTemplate.delete("products:category:" + event.getCategoryId());
    }
}
```

**Manual Invalidation**:
```java
@Service
class ManualCacheService {
    
    @CacheEvict(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        return repository.save(product);
    }
    
    @CacheEvict(value = "products", allEntries = true)
    public void clearAllProductsCache() {
        // All product caches cleared
    }
}
```

---

## 3. Database Scaling Techniques

### 3.1 Read Replicas

**Architecture**:
```
                    ┌──────────────┐
                    │   Primary    │
                    │  (Writes)    │
                    └──────┬───────┘
                           │
            ┌──────────────┼──────────────┐
            ▼              ▼              ▼
    ┌──────────┐   ┌──────────┐   ┌──────────┐
    │ Replica 1 │   │ Replica 2 │   │ Replica 3 │
    │  (Reads)  │   │  (Reads)  │   │  (Reads)  │
    └──────────┘   └──────────┘   └──────────┘
```

**Implementation**:
```java
@Configuration
class ReadReplicaConfig {
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://primary-db:5432/mydb")
            .driverClassName("org.postgresql.Driver")
            .username("user")
            .password("password")
            .build();
    }
    
    @Bean
    public DataSource replicaDataSource1() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://replica1-db:5432/mydb")
            .driverClassName("org.postgresql.Driver")
            .username("user")
            .password("password")
            .build();
    }
    
    @Bean
    public DataSource replicaDataSource2() {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://replica2-db:5432/mydb")
            .driverClassName("org.postgresql.Driver")
            .username("user")
            .password("password")
            .build();
    }
    
    @Bean
    public DataSource routingDataSource() {
        ReplicationRoutingDataSource routing = new ReplicationRoutingDataSource();
        
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("primary", primaryDataSource());
        dataSourceMap.put("replica1", replicaDataSource1());
        dataSourceMap.put("replica2", replicaDataSource2());
        
        routing.setTargetDataSources(dataSourceMap);
        routing.setDefaultTargetDataSource(primaryDataSource());
        
        return routing;
    }
}

class ReplicationRoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return TransactionSynchronizationManager.isCurrentTransactionReadOnly()
            ? getReplicaKey()
            : "primary";
    }
    
    private String getReplicaKey() {
        // Round-robin or least connections
        int replica = ThreadLocalRandom.current().nextInt(1, 3);
        return "replica" + replica;
    }
}
```

### 3.2 Database Sharding

**Sharding Strategies**:

**1. Range-Based Sharding**:
```java
class RangeBasedSharding {
    private Map<String, DataSource> shards = new HashMap<>();
    
    public DataSource getShard(String userId) {
        // Shard based on user ID range
        int hash = userId.hashCode();
        int shardIndex = Math.abs(hash) % 4; // 4 shards
        
        return shards.get("shard" + shardIndex);
    }
}
```

**2. Hash-Based Sharding**:
```java
class HashBasedSharding {
    private List<DataSource> shards;
    
    public DataSource getShard(String key) {
        // Consistent hashing
        int hash = key.hashCode();
        int shardIndex = Math.abs(hash) % shards.size();
        
        return shards.get(shardIndex);
    }
}
```

**3. Directory-Based Sharding**:
```java
class DirectoryBasedSharding {
    private Map<String, DataSource> shardDirectory = new HashMap<>();
    
    public DataSource getShard(String tenantId) {
        // Lookup shard from directory
        return shardDirectory.get(tenantId);
    }
    
    public void addShardMapping(String tenantId, DataSource shard) {
        shardDirectory.put(tenantId, shard);
    }
}
```

**Complete Sharding Implementation**:
```java
@Configuration
class ShardingConfig {
    
    @Bean
    public DataSource shardedDataSource() {
        Map<String, DataSource> dataSourceMap = new HashMap<>();
        
        // Create shards
        dataSourceMap.put("shard0", createDataSource("shard0-db"));
        dataSourceMap.put("shard1", createDataSource("shard1-db"));
        dataSourceMap.put("shard2", createDataSource("shard2-db"));
        dataSourceMap.put("shard3", createDataSource("shard3-db"));
        
        ShardingDataSource shardingDataSource = new ShardingDataSource();
        shardingDataSource.setTargetDataSources(dataSourceMap);
        shardingDataSource.setDefaultTargetDataSource(dataSourceMap.get("shard0"));
        
        return shardingDataSource;
    }
    
    private DataSource createDataSource(String host) {
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://" + host + ":5432/mydb")
            .username("user")
            .password("password")
            .build();
    }
}

class ShardingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        String shardKey = ShardContext.getCurrentShardKey();
        return calculateShard(shardKey);
    }
    
    private String calculateShard(String key) {
        // Consistent hashing
        int hash = key.hashCode();
        int shardIndex = Math.abs(hash) % 4;
        return "shard" + shardIndex;
    }
}

class ShardContext {
    private static final ThreadLocal<String> shardKey = new ThreadLocal<>();
    
    public static void setCurrentShardKey(String key) {
        shardKey.set(key);
    }
    
    public static String getCurrentShardKey() {
        return shardKey.get();
    }
    
    public static void clear() {
        shardKey.remove();
    }
}

// Usage
@Service
class ShardedUserService {
    
    @Autowired
    private UserRepository userRepository;
    
    public User getUser(String userId) {
        // Set shard key based on user ID
        ShardContext.setCurrentShardKey(userId);
        try {
            return userRepository.findById(userId);
        } finally {
            ShardContext.clear();
        }
    }
}
```

### 3.3 Database Partitioning

**Horizontal Partitioning**:
```java
/**
 * Horizontal Partitioning Example:
 * 
 * Partition users table by date:
 * - users_2024_01
 * - users_2024_02
 * - users_2024_03
 */

@Service
class PartitionedUserService {
    
    public User getUser(String userId, LocalDate date) {
        String tableName = "users_" + date.format(DateTimeFormatter.ofPattern("yyyy_MM"));
        
        // Query specific partition
        return userRepository.findInPartition(userId, tableName);
    }
}
```

**Vertical Partitioning**:
```java
/**
 * Vertical Partitioning Example:
 * 
 * Split user table:
 * - users (id, name, email) - Frequently accessed
 * - user_profiles (id, bio, preferences) - Less frequently accessed
 */

@Entity
@Table(name = "users")
class User {
    @Id
    private String id;
    private String name;
    private String email;
}

@Entity
@Table(name = "user_profiles")
class UserProfile {
    @Id
    private String userId;
    private String bio;
    private String preferences;
}
```

---

## 4. Sharding Strategies

### 4.1 Consistent Hashing

```java
import java.security.MessageDigest;
import java.util.*;

class ConsistentHash {
    private final TreeMap<Long, String> ring = new TreeMap<>();
    private final int numberOfReplicas;
    
    public ConsistentHash(int numberOfReplicas, List<String> nodes) {
        this.numberOfReplicas = numberOfReplicas;
        for (String node : nodes) {
            addNode(node);
        }
    }
    
    public void addNode(String node) {
        for (int i = 0; i < numberOfReplicas; i++) {
            long hash = hash(node + ":" + i);
            ring.put(hash, node);
        }
    }
    
    public void removeNode(String node) {
        for (int i = 0; i < numberOfReplicas; i++) {
            long hash = hash(node + ":" + i);
            ring.remove(hash);
        }
    }
    
    public String getNode(String key) {
        if (ring.isEmpty()) {
            return null;
        }
        
        long hash = hash(key);
        Map.Entry<Long, String> entry = ring.ceilingEntry(hash);
        
        if (entry == null) {
            // Wrap around to first node
            entry = ring.firstEntry();
        }
        
        return entry.getValue();
    }
    
    private long hash(String key) {
        try {
            MessageDigest md = MessageDigest.getInstance("MD5");
            byte[] digest = md.digest(key.getBytes());
            return ((long) (digest[0] & 0xFF) << 24) |
                   ((long) (digest[1] & 0xFF) << 16) |
                   ((long) (digest[2] & 0xFF) << 8) |
                   ((long) (digest[3] & 0xFF));
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }
}

// Usage
public class ConsistentHashingExample {
    public static void main(String[] args) {
        List<String> nodes = Arrays.asList("node1", "node2", "node3", "node4");
        ConsistentHash consistentHash = new ConsistentHash(3, nodes);
        
        // Get node for key
        String node = consistentHash.getNode("user123");
        System.out.println("Key 'user123' maps to: " + node);
        
        // Add new node
        consistentHash.addNode("node5");
        
        // Remove node
        consistentHash.removeNode("node2");
    }
}
```

### 4.2 Shard Key Selection

```java
/**
 * Shard Key Selection Guidelines:
 * 
 * Good Shard Keys:
 * - High cardinality (many unique values)
 * - Even distribution
 * - Frequently used in queries
 * - Doesn't change frequently
 * 
 * Bad Shard Keys:
 * - Low cardinality (few unique values)
 * - Skewed distribution
 * - Rarely used in queries
 * - Changes frequently
 */

class ShardKeySelector {
    
    // Good: User ID (high cardinality, even distribution)
    public String getShardKeyForUser(String userId) {
        return userId; // Use directly
    }
    
    // Good: Composite key (user_id + order_id)
    public String getShardKeyForOrder(String userId, String orderId) {
        return userId + ":" + orderId; // User-based sharding
    }
    
    // Bad: Status field (low cardinality)
    // Don't shard by: "active", "inactive", "pending"
    
    // Good: Geographic sharding
    public String getShardKeyByRegion(String region) {
        return region; // If queries are region-based
    }
}
```

---

## 5. Practical Examples

### 5.1 E-Commerce Platform Scaling

**Architecture**:
```java
/**
 * E-Commerce Platform Architecture:
 * 
 * Frontend:
 *   - CDN for static assets
 *   - Browser caching
 * 
 * Application Layer:
 *   - Load balanced application servers
 *   - Redis cache layer
 *   - Session store
 * 
 * Data Layer:
 *   - Primary database (writes)
 *   - Read replicas (reads)
 *   - Search index (Elasticsearch)
 * 
 * Message Queue:
 *   - Order processing
 *   - Email notifications
 *   - Inventory updates
 */
```

**Implementation**:
```java
// Product Service with Caching and Read Replicas
@Service
class ScalableProductService {
    
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private RedisTemplate<String, Product> redisTemplate;
    
    @Autowired
    private ElasticsearchTemplate elasticsearchTemplate;
    
    // Read from cache, fallback to read replica
    @Transactional(readOnly = true)
    public Product getProduct(String id) {
        // L1: Check cache
        String cacheKey = "product:" + id;
        Product cached = redisTemplate.opsForValue().get(cacheKey);
        if (cached != null) {
            return cached;
        }
        
        // L2: Read from database (read replica)
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("Product not found"));
        
        // L3: Write to cache
        redisTemplate.opsForValue().set(cacheKey, product, Duration.ofHours(1));
        
        return product;
    }
    
    // Search with Elasticsearch
    public List<Product> searchProducts(String query) {
        // Search in Elasticsearch (fast, scalable)
        return elasticsearchTemplate.search(query, Product.class);
    }
    
    // Write to primary database
    @Transactional
    public Product createProduct(Product product) {
        // Write to primary database
        Product saved = productRepository.save(product);
        
        // Invalidate cache
        redisTemplate.delete("product:" + saved.getId());
        
        // Index in Elasticsearch (async)
        elasticsearchTemplate.save(saved);
        
        return saved;
    }
}
```

### 5.2 Social Media Platform Scaling

**Architecture**:
```java
/**
 * Social Media Platform Architecture:
 * 
 * Timeline Service:
 *   - Pre-computed timelines (Redis)
 *   - Fan-out on write
 *   - Fan-out on read (hybrid)
 * 
 * Feed Generation:
 *   - Write: Fan-out to followers' timelines
 *   - Read: Aggregate from followed users
 */
```

**Fan-Out on Write**:
```java
@Service
class TimelineService {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    @Autowired
    private KafkaTemplate<String, PostEvent> kafkaTemplate;
    
    public void createPost(String userId, Post post) {
        // 1. Save post
        postRepository.save(post);
        
        // 2. Get followers
        List<String> followers = userServiceClient.getFollowers(userId);
        
        // 3. Fan-out: Add to each follower's timeline
        for (String followerId : followers) {
            String timelineKey = "timeline:" + followerId;
            redisTemplate.opsForList().leftPush(timelineKey, post.getId());
            redisTemplate.opsForList().trim(timelineKey, 0, 999); // Keep last 1000
        }
        
        // 4. Publish event for async processing
        kafkaTemplate.send("post-events", new PostEvent(userId, post.getId()));
    }
    
    public List<Post> getTimeline(String userId) {
        String timelineKey = "timeline:" + userId;
        List<String> postIds = redisTemplate.opsForList()
            .range(timelineKey, 0, 99); // Get latest 100
        
        // Fetch posts
        return postIds.stream()
            .map(postId -> postRepository.findById(postId))
            .filter(Optional::isPresent)
            .map(Optional::get)
            .collect(Collectors.toList());
    }
}
```

---

## Summary: Part 2

### Key Concepts

1. **Multi-Layer Caching**: Browser → CDN → Application → Database
2. **Cache Patterns**: Cache-Aside, Write-Through, Write-Behind
3. **Database Scaling**: Read Replicas, Sharding, Partitioning
4. **Sharding Strategies**: Range, Hash, Directory, Consistent Hashing

### Next Steps

**Part 3** will cover:
- Message Queues and Event Streaming
- Microservices Communication
- API Gateway Patterns
- Service Mesh

---

**Master caching and database scaling for high-performance systems!**

