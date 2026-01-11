# System Architecture for Scale: In-Depth Guide

## Part 4: CDN, Edge Computing, Monitoring, and Performance Optimization

---

## Table of Contents

1. [CDN and Edge Computing](#1-cdn-and-edge-computing)
2. [Monitoring and Observability](#2-monitoring-and-observability)
3. [Performance Optimization](#3-performance-optimization)
4. [Disaster Recovery and High Availability](#4-disaster-recovery-and-high-availability)
5. [Practical Examples](#5-practical-examples)

---

## 1. CDN and Edge Computing

### 1.1 CDN Architecture

**CDN Structure**:
```
Origin Server
     │
     ▼
Edge Locations (Global)
  ├─ US East
  ├─ US West
  ├─ Europe
  ├─ Asia
  └─ ...
```

**How CDN Works**:
```java
/**
 * CDN Flow:
 * 
 * 1. First Request:
 *    Client → Edge Location (cache miss) → Origin Server
 *    Edge Location caches response
 * 
 * 2. Subsequent Requests:
 *    Client → Edge Location (cache hit) → Client
 *    (No origin server call)
 */
```

### 1.2 CDN Implementation

**CloudFront Configuration**:
```java
@Configuration
class CDNConfig {
    
    /**
     * CDN Configuration:
     * 
     * 1. Static Assets:
     *    - CSS, JS, Images
     *    - Long TTL (1 year)
     *    - Cache-Control: public, max-age=31536000
     * 
     * 2. Dynamic Content:
     *    - API responses
     *    - Short TTL (5 minutes)
     *    - Cache-Control: public, max-age=300
     * 
     * 3. Private Content:
     *    - User-specific content
     *    - Signed URLs
     *    - No caching
     */
}

// Static Asset Serving
@RestController
class StaticAssetController {
    
    @GetMapping("/assets/{filename}")
    public ResponseEntity<Resource> getAsset(
            @PathVariable String filename,
            HttpServletRequest request) {
        
        Resource resource = new ClassPathResource("static/" + filename);
        
        // Set cache headers
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(1, TimeUnit.YEARS))
            .eTag(calculateETag(resource))
            .body(resource);
    }
    
    private String calculateETag(Resource resource) {
        // Calculate ETag based on file content
        return String.valueOf(resource.hashCode());
    }
}

// Dynamic Content with CDN
@RestController
class ProductController {
    
    @GetMapping("/api/products/{id}")
    public ResponseEntity<Product> getProduct(
            @PathVariable String id,
            @RequestHeader(value = "If-None-Match", required = false) String ifNoneMatch) {
        
        Product product = productService.getProduct(id);
        String etag = product.getVersion().toString();
        
        // Return 304 if not modified
        if (etag.equals(ifNoneMatch)) {
            return ResponseEntity.status(HttpStatus.NOT_MODIFIED)
                .eTag(etag)
                .build();
        }
        
        // Set cache headers for CDN
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(5, TimeUnit.MINUTES))
            .eTag(etag)
            .body(product);
    }
}
```

### 1.3 Edge Computing Patterns

**Lambda@Edge Example**:
```javascript
// CloudFront Lambda@Edge Function
exports.handler = async (event) => {
    const request = event.Records[0].cf.request;
    const headers = request.headers;
    
    // A/B Testing at Edge
    const userAgent = headers['user-agent'][0].value;
    const variant = userAgent.includes('Mobile') ? 'mobile' : 'desktop';
    
    // Route to different origins based on device
    if (variant === 'mobile') {
        request.origin = {
            custom: {
                domainName: 'mobile-api.example.com',
                port: 443,
                protocol: 'https'
            }
        };
    }
    
    return request;
};
```

**Edge Caching Strategy**:
```java
@Service
class EdgeCacheService {
    
    /**
     * Edge Caching Strategy:
     * 
     * 1. Static Content:
     *    - Cache at edge (CDN)
     *    - Long TTL
     *    - No origin calls
     * 
     * 2. Dynamic Content:
     *    - Cache at edge with short TTL
     *    - Stale-while-revalidate
     *    - Origin fallback
     * 
     * 3. Personalized Content:
     *    - No edge caching
     *    - Direct origin calls
     *    - Use signed URLs
     */
    
    public ResponseEntity<Product> getProduct(String id, String userId) {
        Product product = productService.getProduct(id);
        
        // Check if personalized
        if (isPersonalizedContent(id, userId)) {
            // No caching for personalized content
            return ResponseEntity.ok()
                .cacheControl(CacheControl.noCache())
                .body(product);
        }
        
        // Cache at edge
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(10, TimeUnit.MINUTES)
                .staleWhileRevalidate(1, TimeUnit.HOURS))
            .body(product);
    }
    
    private boolean isPersonalizedContent(String productId, String userId) {
        // Check if content is user-specific
        return productService.hasUserSpecificPricing(productId, userId);
    }
}
```

---

## 2. Monitoring and Observability

### 2.1 Three Pillars of Observability

**1. Metrics**
```java
@Service
class MetricsService {
    
    private final MeterRegistry meterRegistry;
    private final Counter orderCounter;
    private final Timer orderProcessingTimer;
    private final Gauge activeUsersGauge;
    
    public MetricsService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
        this.orderCounter = Counter.builder("orders.created")
            .description("Total orders created")
            .register(meterRegistry);
        this.orderProcessingTimer = Timer.builder("orders.processing.time")
            .description("Order processing time")
            .register(meterRegistry);
        this.activeUsersGauge = Gauge.builder("users.active")
            .description("Active users")
            .register(meterRegistry, this, MetricsService::getActiveUsers);
    }
    
    public void recordOrderCreated() {
        orderCounter.increment();
    }
    
    public void recordOrderProcessingTime(Duration duration) {
        orderProcessingTimer.record(duration);
    }
    
    private double getActiveUsers() {
        return userService.getActiveUserCount();
    }
}

// Usage
@Service
class OrderService {
    
    @Autowired
    private MetricsService metricsService;
    
    public Order createOrder(OrderRequest request) {
        Timer.Sample sample = Timer.start(metricsService.getMeterRegistry());
        
        try {
            Order order = processOrder(request);
            metricsService.recordOrderCreated();
            return order;
        } finally {
            sample.stop(metricsService.getOrderProcessingTimer());
        }
    }
}
```

**2. Logging**
```java
@Slf4j
@Service
class LoggingService {
    
    private static final Logger logger = LoggerFactory.getLogger(LoggingService.class);
    
    public void logOrderCreated(Order order) {
        // Structured logging
        MDC.put("orderId", order.getId());
        MDC.put("userId", order.getUserId());
        MDC.put("total", String.valueOf(order.getTotal()));
        
        logger.info("Order created successfully", 
            kv("orderId", order.getId()),
            kv("userId", order.getUserId()),
            kv("total", order.getTotal()));
        
        MDC.clear();
    }
    
    public void logError(String operation, Exception e) {
        logger.error("Error in {}: {}", operation, e.getMessage(), e);
    }
}

// Logging Configuration
@Configuration
class LoggingConfig {
    
    @Bean
    public PatternLayoutEncoder encoder() {
        PatternLayoutEncoder encoder = new PatternLayoutEncoder();
        encoder.setPattern("%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n");
        return encoder;
    }
}
```

**3. Tracing**
```java
@Service
class TracingService {
    
    @Autowired
    private Tracer tracer;
    
    public Order createOrder(OrderRequest request) {
        Span span = tracer.nextSpan()
            .name("create-order")
            .tag("userId", request.getUserId())
            .start();
        
        try (Tracer.SpanInScope ws = tracer.withSpanInScope(span)) {
            // Process order
            Order order = processOrder(request);
            
            span.tag("orderId", order.getId());
            span.tag("status", "success");
            
            return order;
        } catch (Exception e) {
            span.tag("error", true);
            span.tag("error.message", e.getMessage());
            throw e;
        } finally {
            span.end();
        }
    }
    
    // Distributed tracing across services
    public void callExternalService(String serviceName, String endpoint) {
        Span span = tracer.nextSpan()
            .name("call-" + serviceName)
            .tag("service", serviceName)
            .tag("endpoint", endpoint)
            .start();
        
        try (Tracer.SpanInScope ws = tracer.withSpanInScope(span)) {
            // Make HTTP call
            restTemplate.getForObject(endpoint, String.class);
        } finally {
            span.end();
        }
    }
}
```

### 2.2 Distributed Tracing

**Zipkin Integration**:
```java
@Configuration
class TracingConfig {
    
    @Bean
    public Sender sender() {
        return OkHttpSender.create("http://zipkin:9411/api/v2/spans");
    }
    
    @Bean
    public AsyncReporter<Span> spanReporter() {
        return AsyncReporter.create(sender());
    }
    
    @Bean
    public Tracing tracing() {
        return Tracing.newBuilder()
            .localServiceName("order-service")
            .spanReporter(spanReporter())
            .sampler(Sampler.create(1.0f)) // 100% sampling
            .build();
    }
}
```

### 2.3 Health Checks

```java
@Component
class HealthIndicator implements HealthIndicator {
    
    @Autowired
    private DatabaseHealthChecker databaseHealthChecker;
    
    @Autowired
    private RedisHealthChecker redisHealthChecker;
    
    @Override
    public Health health() {
        Health.Builder builder = new Health.Builder();
        
        // Check database
        boolean dbHealthy = databaseHealthChecker.isHealthy();
        builder.withDetail("database", dbHealthy ? "UP" : "DOWN");
        
        // Check Redis
        boolean redisHealthy = redisHealthChecker.isHealthy();
        builder.withDetail("redis", redisHealthy ? "UP" : "DOWN");
        
        if (dbHealthy && redisHealthy) {
            return builder.up().build();
        } else {
            return builder.down().build();
        }
    }
}

@Component
class DatabaseHealthChecker {
    
    @Autowired
    private DataSource dataSource;
    
    public boolean isHealthy() {
        try {
            dataSource.getConnection().isValid(1);
            return true;
        } catch (SQLException e) {
            return false;
        }
    }
}

@Component
class RedisHealthChecker {
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    public boolean isHealthy() {
        try {
            redisTemplate.getConnectionFactory().getConnection().ping();
            return true;
        } catch (Exception e) {
            return false;
        }
    }
}
```

---

## 3. Performance Optimization

### 3.1 Database Query Optimization

**Connection Pooling**:
```java
@Configuration
class DatabaseConfig {
    
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:postgresql://localhost:5432/mydb");
        config.setUsername("user");
        config.setPassword("password");
        
        // Connection pool settings
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        
        return new HikariDataSource(config);
    }
}
```

**Query Optimization**:
```java
@Repository
class OptimizedUserRepository {
    
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    // Use indexes
    public List<User> findActiveUsers() {
        // Index on status column
        return jdbcTemplate.query(
            "SELECT * FROM users WHERE status = 'ACTIVE'",
            new UserRowMapper()
        );
    }
    
    // Batch operations
    public void saveAll(List<User> users) {
        jdbcTemplate.batchUpdate(
            "INSERT INTO users (id, name, email) VALUES (?, ?, ?)",
            users,
            100, // Batch size
            (ps, user) -> {
                ps.setString(1, user.getId());
                ps.setString(2, user.getName());
                ps.setString(3, user.getEmail());
            }
        );
    }
    
    // Pagination
    public Page<User> findUsers(Pageable pageable) {
        String countSql = "SELECT COUNT(*) FROM users";
        int total = jdbcTemplate.queryForObject(countSql, Integer.class);
        
        String dataSql = "SELECT * FROM users LIMIT ? OFFSET ?";
        List<User> users = jdbcTemplate.query(
            dataSql,
            new Object[]{pageable.getPageSize(), pageable.getOffset()},
            new UserRowMapper()
        );
        
        return new PageImpl<>(users, pageable, total);
    }
}
```

### 3.2 Async Processing

```java
@Service
class AsyncOrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Async("orderProcessingExecutor")
    public CompletableFuture<Order> processOrderAsync(OrderRequest request) {
        // Long-running operation
        Order order = processOrder(request);
        return CompletableFuture.completedFuture(order);
    }
    
    @Async("emailExecutor")
    public CompletableFuture<Void> sendEmailAsync(String email, String subject, String body) {
        emailService.sendEmail(email, subject, body);
        return CompletableFuture.completedFuture(null);
    }
}

@Configuration
@EnableAsync
class AsyncConfig {
    
    @Bean(name = "orderProcessingExecutor")
    public Executor orderProcessingExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("order-processing-");
        executor.initialize();
        return executor;
    }
    
    @Bean(name = "emailExecutor")
    public Executor emailExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("email-");
        executor.initialize();
        return executor;
    }
}
```

### 3.3 Response Compression

```java
@Configuration
class CompressionConfig {
    
    @Bean
    public FilterRegistrationBean<GzipFilter> gzipFilter() {
        FilterRegistrationBean<GzipFilter> registration = new FilterRegistrationBean<>();
        registration.setFilter(new GzipFilter());
        registration.addUrlPatterns("/*");
        registration.setOrder(1);
        return registration;
    }
}

class GzipFilter implements Filter {
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        String acceptEncoding = httpRequest.getHeader("Accept-Encoding");
        
        if (acceptEncoding != null && acceptEncoding.contains("gzip")) {
            GzipResponseWrapper wrappedResponse = new GzipResponseWrapper(httpResponse);
            chain.doFilter(request, wrappedResponse);
            wrappedResponse.finish();
        } else {
            chain.doFilter(request, response);
        }
    }
}
```

### 3.4 Database Connection Pooling

```java
@Configuration
class ConnectionPoolConfig {
    
    @Bean
    public DataSource dataSource() {
        HikariConfig config = new HikariConfig();
        
        // Connection pool settings
        config.setMaximumPoolSize(20);
        config.setMinimumIdle(5);
        config.setConnectionTimeout(30000);
        config.setIdleTimeout(600000);
        config.setMaxLifetime(1800000);
        config.setLeakDetectionThreshold(60000);
        
        // Performance settings
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        
        return new HikariDataSource(config);
    }
}
```

---

## 4. Disaster Recovery and High Availability

### 4.1 Multi-Region Deployment

**Architecture**:
```
Region 1 (Primary)          Region 2 (Secondary)
┌──────────────┐          ┌──────────────┐
│  App Servers │          │  App Servers │
│  Database    │◄────────┤  Database    │ (Replication)
│  Cache       │          │  Cache       │
└──────────────┘          └──────────────┘
```

**Implementation**:
```java
@Configuration
class MultiRegionConfig {
    
    @Bean
    @Primary
    public DataSource primaryRegionDataSource() {
        return createDataSource("primary-region-db");
    }
    
    @Bean
    public DataSource secondaryRegionDataSource() {
        return createDataSource("secondary-region-db");
    }
    
    @Bean
    public DataSource routingDataSource() {
        MultiRegionRoutingDataSource routing = new MultiRegionRoutingDataSource();
        
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("primary", primaryRegionDataSource());
        dataSourceMap.put("secondary", secondaryRegionDataSource());
        
        routing.setTargetDataSources(dataSourceMap);
        routing.setDefaultTargetDataSource(primaryRegionDataSource());
        
        return routing;
    }
}

class MultiRegionRoutingDataSource extends AbstractRoutingDataSource {
    
    @Override
    protected Object determineCurrentLookupKey() {
        // Check if primary region is healthy
        if (isPrimaryRegionHealthy()) {
            return "primary";
        } else {
            // Failover to secondary
            return "secondary";
        }
    }
    
    private boolean isPrimaryRegionHealthy() {
        // Health check logic
        return healthCheckService.isPrimaryRegionHealthy();
    }
}
```

### 4.2 Circuit Breaker Pattern

```java
@Service
class CircuitBreakerService {
    
    @Autowired
    private CircuitBreakerRegistry circuitBreakerRegistry;
    
    private CircuitBreaker circuitBreaker;
    
    @PostConstruct
    public void init() {
        circuitBreaker = circuitBreakerRegistry.circuitBreaker("external-service", 
            CircuitBreakerConfig.custom()
                .failureRateThreshold(50)
                .waitDurationInOpenState(Duration.ofSeconds(30))
                .slidingWindowSize(10)
                .build());
    }
    
    public String callExternalService() {
        return circuitBreaker.executeSupplier(() -> {
            // Call external service
            return restTemplate.getForObject("http://external-service/api", String.class);
        });
    }
    
    public String callWithFallback() {
        return circuitBreaker.executeSupplier(() -> {
            return restTemplate.getForObject("http://external-service/api", String.class);
        }, throwable -> {
            // Fallback logic
            return "Fallback response";
        });
    }
}
```

### 4.3 Retry Pattern

```java
@Service
class RetryService {
    
    @Autowired
    private RetryRegistry retryRegistry;
    
    private Retry retry;
    
    @PostConstruct
    public void init() {
        retry = retryRegistry.retry("external-service",
            RetryConfig.custom()
                .maxAttempts(3)
                .waitDuration(Duration.ofSeconds(1))
                .retryExceptions(IOException.class, TimeoutException.class)
                .build());
    }
    
    public String callWithRetry() {
        return retry.executeSupplier(() -> {
            return restTemplate.getForObject("http://external-service/api", String.class);
        });
    }
}
```

### 4.4 Bulkhead Pattern

```java
@Configuration
class BulkheadConfig {
    
    @Bean
    public ThreadPoolBulkhead orderProcessingBulkhead() {
        return ThreadPoolBulkhead.of("order-processing",
            ThreadPoolBulkheadConfig.custom()
                .maxThreadPoolSize(10)
                .coreThreadPoolSize(5)
                .queueCapacity(20)
                .build());
    }
    
    @Bean
    public ThreadPoolBulkhead emailBulkhead() {
        return ThreadPoolBulkhead.of("email-processing",
            ThreadPoolBulkheadConfig.custom()
                .maxThreadPoolSize(5)
                .coreThreadPoolSize(2)
                .queueCapacity(10)
                .build());
    }
}

@Service
class BulkheadService {
    
    @Autowired
    private ThreadPoolBulkhead orderProcessingBulkhead;
    
    @Autowired
    private ThreadPoolBulkhead emailBulkhead;
    
    public CompletableFuture<Order> processOrder(Order order) {
        return orderProcessingBulkhead.executeSupplier(() -> {
            return processOrderLogic(order);
        });
    }
    
    public CompletableFuture<Void> sendEmail(String email) {
        return emailBulkhead.executeSupplier(() -> {
            emailService.send(email);
            return null;
        });
    }
}
```

---

## 5. Practical Examples

### 5.1 Complete Scalable Architecture

**Full Stack Architecture**:
```java
/**
 * Complete Scalable Architecture:
 * 
 * 1. CDN Layer:
 *    - CloudFront/Cloudflare
 *    - Static assets caching
 *    - Edge locations
 * 
 * 2. Load Balancer:
 *    - Application Load Balancer
 *    - Health checks
 *    - SSL termination
 * 
 * 3. Application Layer:
 *    - Auto-scaling groups
 *    - Multiple availability zones
 *    - Stateless design
 * 
 * 4. Caching Layer:
 *    - Redis cluster
 *    - Multi-layer caching
 *    - Cache invalidation
 * 
 * 5. Database Layer:
 *    - Primary database (writes)
 *    - Read replicas (reads)
 *    - Cross-region replication
 * 
 * 6. Message Queue:
 *    - Kafka cluster
 *    - Event streaming
 *    - Async processing
 * 
 * 7. Monitoring:
 *    - CloudWatch/Prometheus
 *    - Distributed tracing
 *    - Log aggregation
 */
```

### 5.2 High-Performance API

```java
@RestController
@RequestMapping("/api/v1")
class HighPerformanceController {
    
    @Autowired
    private ProductService productService;
    
    @GetMapping("/products/{id}")
    public ResponseEntity<Product> getProduct(
            @PathVariable String id,
            @RequestHeader(value = "If-None-Match", required = false) String ifNoneMatch) {
        
        // Check cache first
        Product product = productService.getProduct(id);
        String etag = product.getVersion().toString();
        
        // ETag validation
        if (etag.equals(ifNoneMatch)) {
            return ResponseEntity.status(HttpStatus.NOT_MODIFIED)
                .eTag(etag)
                .build();
        }
        
        // Return with cache headers
        return ResponseEntity.ok()
            .cacheControl(CacheControl.maxAge(10, TimeUnit.MINUTES)
                .staleWhileRevalidate(1, TimeUnit.HOURS))
            .eTag(etag)
            .body(product);
    }
    
    @GetMapping("/products")
    public ResponseEntity<List<Product>> getProducts(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        
        Pageable pageable = PageRequest.of(page, size);
        Page<Product> products = productService.getProducts(pageable);
        
        return ResponseEntity.ok()
            .header("X-Total-Count", String.valueOf(products.getTotalElements()))
            .header("X-Total-Pages", String.valueOf(products.getTotalPages()))
            .body(products.getContent());
    }
}
```

---

## Summary: Part 4

### Key Concepts

1. **CDN**: Edge caching for static and dynamic content
2. **Monitoring**: Metrics, Logging, Tracing (Three Pillars)
3. **Performance**: Query optimization, async processing, compression
4. **High Availability**: Multi-region, circuit breakers, retries, bulkheads

### Next Steps

**Part 5** will cover:
- Real-World Case Studies
- Scaling Strategies by Use Case
- Cost Optimization
- Best Practices Summary

---

**Master CDN, monitoring, and high availability for production-ready systems!**

