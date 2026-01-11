# System Architecture for Scale: In-Depth Guide

## Part 1: Fundamentals of Scalability and Architecture Patterns

---

## Table of Contents

1. [Scalability Fundamentals](#1-scalability-fundamentals)
2. [Scaling Dimensions](#2-scaling-dimensions)
3. [Architecture Patterns for Scale](#3-architecture-patterns-for-scale)
4. [Load Balancing Strategies](#4-load-balancing-strategies)
5. [Practical Examples](#5-practical-examples)

---

## 1. Scalability Fundamentals

### 1.1 What is Scalability?

**Scalability** is the ability of a system to handle increased load by adding resources without significant performance degradation.

### 1.2 Key Metrics

```java
/**
 * Key Scalability Metrics:
 * 
 * 1. Throughput: Requests per second (RPS)
 * 2. Latency: Response time (P50, P95, P99)
 * 3. Availability: Uptime percentage (99.9%, 99.99%)
 * 4. Capacity: Maximum load system can handle
 * 5. Resource Utilization: CPU, Memory, Network usage
 */
```

### 1.3 Scalability vs Performance

```java
/**
 * Performance: How fast a system responds (latency)
 * Scalability: How well system handles increased load
 * 
 * A fast system may not be scalable
 * A scalable system may not be the fastest
 * 
 * Goal: Both high performance AND scalability
 */
```

---

## 2. Scaling Dimensions

### 2.1 Vertical Scaling (Scale Up)

**Definition**: Increase resources of a single machine (CPU, RAM, Storage)

**Pros**:
- Simple to implement
- No code changes needed
- Lower complexity

**Cons**:
- Limited by hardware
- Single point of failure
- Expensive at scale

**Example**:
```java
/**
 * Vertical Scaling Example:
 * 
 * Before: 4 CPU cores, 8GB RAM
 * After:  16 CPU cores, 64GB RAM
 * 
 * Same application, more powerful hardware
 */
```

### 2.2 Horizontal Scaling (Scale Out)

**Definition**: Add more machines to handle increased load

**Pros**:
- Unlimited scaling potential
- Better fault tolerance
- Cost-effective

**Cons**:
- Requires stateless design
- More complex architecture
- Network overhead

**Example**:
```java
/**
 * Horizontal Scaling Example:
 * 
 * Before: 1 server handling 1000 req/s
 * After:  10 servers handling 10000 req/s
 * 
 * Application must be designed for horizontal scaling
 */
```

### 2.3 Scaling Decision Matrix

```java
/**
 * When to Scale Vertically:
 * - Small to medium load
 * - Application not designed for horizontal scaling
 * - Quick fix needed
 * - Single instance sufficient
 * 
 * When to Scale Horizontally:
 * - High load expected
 * - Need high availability
 * - Cost-effective scaling
 * - Stateless application
 */
```

---

## 3. Architecture Patterns for Scale

### 3.1 Monolithic Architecture

**Structure**:
```
┌─────────────────────────────────┐
│     Monolithic Application     │
│  ┌──────────────────────────┐ │
│  │  Presentation Layer       │ │
│  ├──────────────────────────┤ │
│  │  Business Logic          │ │
│  ├──────────────────────────┤ │
│  │  Data Access Layer       │ │
│  └──────────────────────────┘ │
└─────────────────────────────────┘
```

**Scaling Approach**:
```java
/**
 * Monolithic Scaling:
 * 
 * 1. Vertical Scaling: Upgrade hardware
 * 2. Horizontal Scaling: Load balance multiple instances
 * 3. Read Replicas: Separate read/write databases
 */
```

**Example: Monolithic Spring Boot Application**
```java
@SpringBootApplication
public class MonolithicApp {
    public static void main(String[] args) {
        SpringApplication.run(MonolithicApp.class, args);
    }
}

// All layers in one application
@RestController
class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/users/{id}")
    public User getUser(@PathVariable String id) {
        return userService.getUser(id);
    }
}

@Service
class UserService {
    @Autowired
    private UserRepository userRepository;
    
    public User getUser(String id) {
        return userRepository.findById(id);
    }
}
```

### 3.2 Microservices Architecture

**Structure**:
```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   User       │  │   Order      │  │  Payment     │
│   Service    │  │   Service    │  │  Service    │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
              ┌──────────▼──────────┐
              │   API Gateway      │
              └─────────────────────┘
```

**Scaling Approach**:
```java
/**
 * Microservices Scaling:
 * 
 * 1. Scale services independently
 * 2. Each service can have different scaling needs
 * 3. Use service mesh for communication
 * 4. Implement circuit breakers
 */
```

**Example: Microservices Implementation**
```java
// User Service
@SpringBootApplication
public class UserServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(UserServiceApplication.class, args);
    }
}

@RestController
@RequestMapping("/api/users")
class UserController {
    @Autowired
    private UserService userService;
    
    @GetMapping("/{id}")
    public User getUser(@PathVariable String id) {
        return userService.getUser(id);
    }
}

// Order Service (Separate application)
@SpringBootApplication
public class OrderServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderServiceApplication.class, args);
    }
}

@RestController
@RequestMapping("/api/orders")
class OrderController {
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private UserServiceClient userServiceClient; // Feign client
    
    @PostMapping
    public Order createOrder(@RequestBody OrderRequest request) {
        // Call user service to validate user
        User user = userServiceClient.getUser(request.getUserId());
        return orderService.createOrder(request, user);
    }
}
```

### 3.3 Service-Oriented Architecture (SOA)

**Structure**:
```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Service A  │  │   Service B  │  │  Service C   │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
              ┌──────────▼──────────┐
              │  Enterprise Service  │
              │        Bus (ESB)      │
              └───────────────────────┘
```

### 3.4 Event-Driven Architecture

**Structure**:
```
┌──────────────┐      ┌──────────────┐      ┌──────────────┐
│   Service A  │─────▶│  Event Bus   │◀─────│  Service B   │
│  (Producer)  │      │  (Kafka/Rabbit)│     │ (Consumer)   │
└──────────────┘      └──────────────┘      └──────────────┘
```

**Example: Event-Driven Microservice**
```java
// Event Producer
@Service
class OrderService {
    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public void createOrder(Order order) {
        // Save order
        orderRepository.save(order);
        
        // Publish event
        OrderEvent event = new OrderEvent(order.getId(), "CREATED");
        kafkaTemplate.send("order-events", event);
    }
}

// Event Consumer
@Component
class PaymentService {
    @KafkaListener(topics = "order-events")
    public void handleOrderEvent(OrderEvent event) {
        if ("CREATED".equals(event.getStatus())) {
            processPayment(event.getOrderId());
        }
    }
    
    private void processPayment(String orderId) {
        // Process payment logic
    }
}
```

---

## 4. Load Balancing Strategies

### 4.1 Load Balancer Types

**1. Application Load Balancer (Layer 7)**
```java
/**
 * Application Load Balancer:
 * - Routes based on HTTP/HTTPS content
 * - Can route based on path, host, headers
 * - Supports SSL termination
 * - Health checks at application level
 */
```

**2. Network Load Balancer (Layer 4)**
```java
/**
 * Network Load Balancer:
 * - Routes based on IP and port
 * - Lower latency
 * - Higher throughput
 * - TCP/UDP load balancing
 */
```

### 4.2 Load Balancing Algorithms

**1. Round Robin**
```java
class RoundRobinLoadBalancer {
    private List<String> servers;
    private int currentIndex = 0;
    
    public RoundRobinLoadBalancer(List<String> servers) {
        this.servers = servers;
    }
    
    public String getNextServer() {
        String server = servers.get(currentIndex);
        currentIndex = (currentIndex + 1) % servers.size();
        return server;
    }
}
```

**2. Least Connections**
```java
class LeastConnectionsLoadBalancer {
    private Map<String, Integer> serverConnections = new HashMap<>();
    
    public String getNextServer() {
        return serverConnections.entrySet().stream()
            .min(Map.Entry.comparingByValue())
            .map(Map.Entry::getKey)
            .orElse(null);
    }
    
    public void incrementConnections(String server) {
        serverConnections.merge(server, 1, Integer::sum);
    }
    
    public void decrementConnections(String server) {
        serverConnections.merge(server, -1, Integer::sum);
    }
}
```

**3. Weighted Round Robin**
```java
class WeightedRoundRobinLoadBalancer {
    private List<ServerWeight> servers;
    private int currentWeight = 0;
    private int currentIndex = -1;
    
    static class ServerWeight {
        String server;
        int weight;
        int currentWeight;
        
        ServerWeight(String server, int weight) {
            this.server = server;
            this.weight = weight;
            this.currentWeight = 0;
        }
    }
    
    public String getNextServer() {
        int totalWeight = servers.stream()
            .mapToInt(s -> s.weight)
            .sum();
        
        while (true) {
            currentIndex = (currentIndex + 1) % servers.size();
            if (currentIndex == 0) {
                currentWeight -= totalWeight;
                if (currentWeight <= 0) {
                    currentWeight = totalWeight;
                }
            }
            
            ServerWeight server = servers.get(currentIndex);
            server.currentWeight += server.weight;
            
            if (server.currentWeight >= currentWeight) {
                return server.server;
            }
        }
    }
}
```

**4. IP Hash**
```java
class IPHashLoadBalancer {
    private List<String> servers;
    
    public String getServerForIP(String clientIP) {
        int hash = clientIP.hashCode();
        int index = Math.abs(hash) % servers.size();
        return servers.get(index);
    }
}
```

### 4.3 Health Checks

```java
@Component
class HealthCheckService {
    private Map<String, Boolean> serverHealth = new ConcurrentHashMap<>();
    
    @Scheduled(fixedRate = 5000) // Check every 5 seconds
    public void checkServerHealth() {
        for (String server : servers) {
            boolean healthy = performHealthCheck(server);
            serverHealth.put(server, healthy);
        }
    }
    
    private boolean performHealthCheck(String server) {
        try {
            RestTemplate restTemplate = new RestTemplate();
            ResponseEntity<String> response = restTemplate.getForEntity(
                server + "/health", String.class);
            return response.getStatusCode().is2xxSuccessful();
        } catch (Exception e) {
            return false;
        }
    }
    
    public List<String> getHealthyServers() {
        return serverHealth.entrySet().stream()
            .filter(Map.Entry::getValue)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
}
```

---

## 5. Practical Examples

### 5.1 Scalable Web Application Architecture

**Architecture Diagram**:
```
                    Internet
                       │
                       ▼
              ┌─────────────────┐
              │  CDN (CloudFront)│
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  Load Balancer  │
              │     (ALB)       │
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  Web Server │ │  Web Server │ │  Web Server │
│   (App 1)   │ │   (App 2)   │ │   (App 3)   │
└──────┬──────┘ └──────┬──────┘ └──────┬──────┘
       │               │               │
       └───────────────┼───────────────┘
                       │
              ┌────────▼────────┐
              │   Cache Layer   │
              │   (Redis)       │
              └────────┬────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  Database   │ │  Database     │ │  Database   │
│  (Primary)   │ │  (Replica 1)  │ │  (Replica 2)│
└─────────────┘ └───────────────┘ └─────────────┘
```

**Implementation Example**:
```java
// Application with caching
@Service
class ProductService {
    @Autowired
    private ProductRepository productRepository;
    
    @Autowired
    private RedisTemplate<String, Product> redisTemplate;
    
    private static final String CACHE_KEY_PREFIX = "product:";
    private static final long CACHE_TTL = 3600; // 1 hour
    
    public Product getProduct(String id) {
        // Check cache first
        String cacheKey = CACHE_KEY_PREFIX + id;
        Product cached = redisTemplate.opsForValue().get(cacheKey);
        
        if (cached != null) {
            return cached; // Cache hit
        }
        
        // Cache miss - fetch from database
        Product product = productRepository.findById(id)
            .orElseThrow(() -> new NotFoundException("Product not found"));
        
        // Store in cache
        redisTemplate.opsForValue().set(cacheKey, product, 
            Duration.ofSeconds(CACHE_TTL));
        
        return product;
    }
    
    public void updateProduct(Product product) {
        // Update database
        productRepository.save(product);
        
        // Invalidate cache
        String cacheKey = CACHE_KEY_PREFIX + product.getId();
        redisTemplate.delete(cacheKey);
    }
}
```

### 5.2 Auto-Scaling Configuration

**Kubernetes Horizontal Pod Autoscaler**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: user-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: user-service
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

**AWS Auto Scaling Group**:
```java
// Auto Scaling Configuration
public class AutoScalingConfig {
    /**
     * Auto Scaling Policy:
     * - Scale up when CPU > 70%
     * - Scale down when CPU < 30%
     * - Min instances: 2
     * - Max instances: 10
     * - Desired capacity: 3
     */
    
    // CloudFormation/CFN equivalent
    /*
    AutoScalingGroup:
      MinSize: 2
      MaxSize: 10
      DesiredCapacity: 3
      TargetTrackingScalingPolicy:
        PredefinedMetricSpecification:
          PredefinedMetricType: ASGAverageCPUUtilization
        TargetValue: 70.0
    */
}
```

### 5.3 Database Scaling Example

**Read Replicas**:
```java
@Configuration
class DatabaseConfig {
    
    @Bean
    @Primary
    public DataSource primaryDataSource() {
        // Primary database (writes)
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://primary-db:5432/mydb")
            .username("user")
            .password("password")
            .build();
    }
    
    @Bean
    public DataSource replicaDataSource() {
        // Read replica (reads only)
        return DataSourceBuilder.create()
            .url("jdbc:postgresql://replica-db:5432/mydb")
            .username("user")
            .password("password")
            .build();
    }
    
    @Bean
    public DataSource routingDataSource() {
        ReplicationRoutingDataSource routingDataSource = 
            new ReplicationRoutingDataSource();
        
        Map<Object, Object> dataSourceMap = new HashMap<>();
        dataSourceMap.put("primary", primaryDataSource());
        dataSourceMap.put("replica", replicaDataSource());
        
        routingDataSource.setTargetDataSources(dataSourceMap);
        routingDataSource.setDefaultTargetDataSource(primaryDataSource());
        
        return routingDataSource;
    }
}

// Routing logic
class ReplicationRoutingDataSource extends AbstractRoutingDataSource {
    @Override
    protected Object determineCurrentLookupKey() {
        return TransactionSynchronizationManager.isCurrentTransactionReadOnly() 
            ? "replica" 
            : "primary";
    }
}

// Usage
@Service
class UserService {
    @Autowired
    private UserRepository userRepository;
    
    @Transactional(readOnly = true) // Routes to replica
    public User getUser(String id) {
        return userRepository.findById(id).orElse(null);
    }
    
    @Transactional // Routes to primary
    public User createUser(User user) {
        return userRepository.save(user);
    }
}
```

---

## Summary: Part 1

### Key Concepts

1. **Scalability**: Ability to handle increased load
2. **Vertical Scaling**: Scale up (more powerful hardware)
3. **Horizontal Scaling**: Scale out (more instances)
4. **Load Balancing**: Distribute load across instances
5. **Architecture Patterns**: Monolithic, Microservices, Event-Driven

### Next Steps

**Part 2** will cover:
- Caching Strategies (Multi-layer caching)
- Database Scaling (Sharding, Partitioning)
- Message Queues and Event Streaming
- CDN and Edge Computing

---

**Master these fundamentals to build scalable systems!**

