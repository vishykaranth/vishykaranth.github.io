# System Architecture for Scale: In-Depth Guide

## Part 3: Message Queues, Event Streaming, and Microservices Communication

---

## Table of Contents

1. [Message Queue Patterns](#1-message-queue-patterns)
2. [Event Streaming Architecture](#2-event-streaming-architecture)
3. [Microservices Communication](#3-microservices-communication)
4. [API Gateway Patterns](#4-api-gateway-patterns)
5. [Practical Examples](#5-practical-examples)

---

## 1. Message Queue Patterns

### 1.1 Why Message Queues?

**Benefits**:
- **Decoupling**: Services don't need to know about each other
- **Scalability**: Handle traffic spikes
- **Reliability**: Guaranteed delivery
- **Asynchronous Processing**: Non-blocking operations

### 1.2 Producer-Consumer Pattern

```java
// Producer
@Service
class OrderProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void createOrder(Order order) {
        // Save order
        orderRepository.save(order);
        
        // Send message to queue
        rabbitTemplate.convertAndSend("order.queue", order);
        
        System.out.println("Order sent to queue: " + order.getId());
    }
}

// Consumer
@Component
class OrderConsumer {
    
    @RabbitListener(queues = "order.queue")
    public void processOrder(Order order) {
        System.out.println("Processing order: " + order.getId());
        
        // Process order
        processPayment(order);
        updateInventory(order);
        sendConfirmation(order);
    }
    
    private void processPayment(Order order) {
        // Payment processing logic
    }
    
    private void updateInventory(Order order) {
        // Inventory update logic
    }
    
    private void sendConfirmation(Order order) {
        // Send confirmation email
    }
}
```

### 1.3 Pub-Sub Pattern

```java
// Publisher
@Service
class EventPublisher {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void publishUserCreatedEvent(User user) {
        UserCreatedEvent event = new UserCreatedEvent(
            user.getId(), 
            user.getName(), 
            user.getEmail()
        );
        
        // Publish to exchange (fanout to all queues)
        rabbitTemplate.convertAndSend("user.events", "", event);
    }
}

// Subscribers
@Component
class EmailServiceSubscriber {
    
    @RabbitListener(queues = "user.created.email")
    public void handleUserCreated(UserCreatedEvent event) {
        System.out.println("Sending welcome email to: " + event.getEmail());
        emailService.sendWelcomeEmail(event.getEmail());
    }
}

@Component
class AnalyticsServiceSubscriber {
    
    @RabbitListener(queues = "user.created.analytics")
    public void handleUserCreated(UserCreatedEvent event) {
        System.out.println("Tracking user creation: " + event.getUserId());
        analyticsService.trackUserCreation(event);
    }
}
```

### 1.4 Request-Reply Pattern

```java
// Requestor
@Service
class PaymentService {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public PaymentResult processPayment(PaymentRequest request) {
        // Send request and wait for reply
        PaymentResult result = (PaymentResult) rabbitTemplate
            .convertSendAndReceive("payment.request.queue", request);
        
        return result;
    }
}

// Replier
@Component
class PaymentProcessor {
    
    @RabbitListener(queues = "payment.request.queue")
    @SendTo("payment.reply.queue")
    public PaymentResult processPayment(PaymentRequest request) {
        // Process payment
        boolean success = processPaymentLogic(request);
        
        return new PaymentResult(success, "Payment processed");
    }
}
```

### 1.5 Dead Letter Queue Pattern

```java
@Configuration
class DeadLetterQueueConfig {
    
    @Bean
    public Queue orderQueue() {
        return QueueBuilder.durable("order.queue")
            .withArgument("x-dead-letter-exchange", "dlx")
            .withArgument("x-dead-letter-routing-key", "order.dlq")
            .build();
    }
    
    @Bean
    public Queue deadLetterQueue() {
        return QueueBuilder.durable("order.dlq").build();
    }
}

// Consumer with retry logic
@Component
class OrderConsumerWithDLQ {
    
    @RabbitListener(queues = "order.queue")
    public void processOrder(Order order, Channel channel, 
                           @Header(AmqpHeaders.DELIVERY_TAG) long tag) {
        try {
            // Process order
            processOrderLogic(order);
            
            // Acknowledge message
            channel.basicAck(tag, false);
        } catch (Exception e) {
            // Reject and send to DLQ
            channel.basicNack(tag, false, false);
        }
    }
    
    // Process messages from DLQ
    @RabbitListener(queues = "order.dlq")
    public void processFailedOrder(Order order) {
        // Log, alert, or retry
        System.err.println("Failed order: " + order.getId());
        // Send alert to admin
    }
}
```

---

## 2. Event Streaming Architecture

### 2.1 Kafka Event Streaming

**Architecture**:
```
Producer → Kafka Topic (Partitions) → Consumers (Consumer Groups)
```

**Producer Example**:
```java
@Service
class KafkaEventProducer {
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            order.getTotal(),
            Instant.now()
        );
        
        // Publish to topic
        kafkaTemplate.send("order-events", order.getId(), event);
    }
    
    public void publishUserEvent(UserEvent event) {
        // Publish with key for partitioning
        kafkaTemplate.send("user-events", event.getUserId(), event);
    }
}
```

**Consumer Example**:
```java
@Component
class KafkaEventConsumer {
    
    @KafkaListener(topics = "order-events", groupId = "order-processors")
    public void handleOrderCreated(
            @Payload OrderCreatedEvent event,
            @Header(KafkaHeaders.RECEIVED_KEY) String key,
            @Header(KafkaHeaders.RECEIVED_PARTITION) int partition) {
        
        System.out.println("Received order event: " + event.getOrderId() + 
                          " from partition: " + partition);
        
        // Process event
        processOrder(event);
    }
    
    @KafkaListener(topics = "order-events", groupId = "email-service")
    public void handleOrderForEmail(OrderCreatedEvent event) {
        // Send confirmation email
        emailService.sendOrderConfirmation(event);
    }
    
    private void processOrder(OrderCreatedEvent event) {
        // Order processing logic
    }
}
```

### 2.2 Event Sourcing Pattern

```java
// Event
abstract class DomainEvent {
    private String aggregateId;
    private Instant occurredAt;
    
    public DomainEvent(String aggregateId) {
        this.aggregateId = aggregateId;
        this.occurredAt = Instant.now();
    }
    
    public String getAggregateId() { return aggregateId; }
    public Instant getOccurredAt() { return occurredAt; }
}

class OrderCreatedEvent extends DomainEvent {
    private String userId;
    private double total;
    
    public OrderCreatedEvent(String orderId, String userId, double total) {
        super(orderId);
        this.userId = userId;
        this.total = total;
    }
}

class OrderPaidEvent extends DomainEvent {
    private String paymentId;
    
    public OrderPaidEvent(String orderId, String paymentId) {
        super(orderId);
        this.paymentId = paymentId;
    }
}

// Event Store
@Service
class EventStore {
    
    @Autowired
    private KafkaTemplate<String, DomainEvent> kafkaTemplate;
    
    public void saveEvent(DomainEvent event) {
        // Store event in Kafka (event log)
        kafkaTemplate.send("event-store", event.getAggregateId(), event);
    }
    
    public List<DomainEvent> getEvents(String aggregateId) {
        // Read events from Kafka for aggregate
        // Reconstruct state by replaying events
        return readEventsFromKafka(aggregateId);
    }
}

// Aggregate
class Order {
    private String id;
    private String userId;
    private double total;
    private String status;
    private List<DomainEvent> uncommittedEvents = new ArrayList<>();
    
    public static Order create(String id, String userId, double total) {
        Order order = new Order();
        order.id = id;
        order.userId = userId;
        order.total = total;
        order.status = "CREATED";
        
        // Record event
        order.recordEvent(new OrderCreatedEvent(id, userId, total));
        
        return order;
    }
    
    public void markAsPaid(String paymentId) {
        this.status = "PAID";
        recordEvent(new OrderPaidEvent(id, paymentId));
    }
    
    private void recordEvent(DomainEvent event) {
        uncommittedEvents.add(event);
    }
    
    public List<DomainEvent> getUncommittedEvents() {
        return new ArrayList<>(uncommittedEvents);
    }
    
    public void markEventsAsCommitted() {
        uncommittedEvents.clear();
    }
    
    // Reconstruct from events
    public static Order fromEvents(List<DomainEvent> events) {
        Order order = new Order();
        for (DomainEvent event : events) {
            order.apply(event);
        }
        return order;
    }
    
    private void apply(DomainEvent event) {
        if (event instanceof OrderCreatedEvent) {
            OrderCreatedEvent e = (OrderCreatedEvent) event;
            this.id = e.getAggregateId();
            this.userId = e.getUserId();
            this.total = e.getTotal();
            this.status = "CREATED";
        } else if (event instanceof OrderPaidEvent) {
            this.status = "PAID";
        }
    }
}
```

### 2.3 CQRS (Command Query Responsibility Segregation)

```java
// Command Side (Write)
@Service
class OrderCommandService {
    
    @Autowired
    private EventStore eventStore;
    
    public void createOrder(CreateOrderCommand command) {
        Order order = Order.create(
            command.getOrderId(),
            command.getUserId(),
            command.getTotal()
        );
        
        // Save events
        for (DomainEvent event : order.getUncommittedEvents()) {
            eventStore.saveEvent(event);
        }
        
        order.markEventsAsCommitted();
    }
    
    public void payOrder(PayOrderCommand command) {
        // Load order from events
        List<DomainEvent> events = eventStore.getEvents(command.getOrderId());
        Order order = Order.fromEvents(events);
        
        // Execute command
        order.markAsPaid(command.getPaymentId());
        
        // Save new events
        for (DomainEvent event : order.getUncommittedEvents()) {
            eventStore.saveEvent(event);
        }
        
        order.markEventsAsCommitted();
    }
}

// Query Side (Read)
@Service
class OrderQueryService {
    
    @Autowired
    private OrderReadRepository readRepository;
    
    public OrderView getOrder(String orderId) {
        // Read from optimized read model
        return readRepository.findById(orderId);
    }
    
    public List<OrderView> getOrdersByUser(String userId) {
        return readRepository.findByUserId(userId);
    }
}

// Read Model Projection
@Component
class OrderProjection {
    
    @Autowired
    private OrderReadRepository readRepository;
    
    @KafkaListener(topics = "event-store", groupId = "order-projection")
    public void projectEvent(DomainEvent event) {
        if (event instanceof OrderCreatedEvent) {
            OrderCreatedEvent e = (OrderCreatedEvent) event;
            OrderView view = new OrderView(
                e.getAggregateId(),
                e.getUserId(),
                e.getTotal(),
                "CREATED"
            );
            readRepository.save(view);
        } else if (event instanceof OrderPaidEvent) {
            OrderPaidEvent e = (OrderPaidEvent) event;
            OrderView view = readRepository.findById(e.getAggregateId());
            if (view != null) {
                view.setStatus("PAID");
                readRepository.save(view);
            }
        }
    }
}
```

---

## 3. Microservices Communication

### 3.1 Synchronous Communication (REST)

```java
// Service Client using Feign
@FeignClient(name = "user-service", url = "${user.service.url}")
interface UserServiceClient {
    
    @GetMapping("/users/{id}")
    User getUser(@PathVariable String id);
    
    @PostMapping("/users")
    User createUser(@RequestBody User user);
    
    @GetMapping("/users/{id}/orders")
    List<Order> getUserOrders(@PathVariable String id);
}

// Service with Circuit Breaker
@Service
class OrderService {
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    @CircuitBreaker(name = "user-service", fallbackMethod = "getUserFallback")
    public Order createOrder(OrderRequest request) {
        // Call user service
        User user = userServiceClient.getUser(request.getUserId());
        
        // Validate and create order
        return orderRepository.save(new Order(user, request));
    }
    
    public Order getUserFallback(OrderRequest request, Exception e) {
        // Fallback logic
        return orderRepository.save(new Order(request));
    }
}
```

### 3.2 Asynchronous Communication (Events)

```java
// Event-Driven Communication
@Service
class OrderService {
    
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    public void createOrder(OrderRequest request) {
        // Create order
        Order order = orderRepository.save(new Order(request));
        
        // Publish event (fire and forget)
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        kafkaTemplate.send("order-events", event);
    }
}

// Other services listen to events
@Component
class InventoryService {
    
    @KafkaListener(topics = "order-events", groupId = "inventory-service")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Update inventory
        inventoryService.reserveItems(event.getOrderId(), event.getItems());
    }
}

@Component
class NotificationService {
    
    @KafkaListener(topics = "order-events", groupId = "notification-service")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Send notification
        notificationService.sendOrderConfirmation(event);
    }
}
```

### 3.3 Service Mesh Pattern

```java
/**
 * Service Mesh Architecture:
 * 
 * Each service has a sidecar proxy that handles:
 * - Service discovery
 * - Load balancing
 * - Circuit breaking
 * - Retries
 * - Security (mTLS)
 * - Observability
 */
```

**Istio Service Mesh Example**:
```yaml
# VirtualService - Routing rules
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: user-service
spec:
  hosts:
  - user-service
  http:
  - match:
    - headers:
        version:
          exact: v2
    route:
    - destination:
        host: user-service
        subset: v2
  - route:
    - destination:
        host: user-service
        subset: v1
      weight: 80
    - destination:
        host: user-service
        subset: v2
      weight: 20

# DestinationRule - Load balancing
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: user-service
spec:
  host: user-service
  trafficPolicy:
    loadBalancer:
      simple: LEAST_CONN
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

---

## 4. API Gateway Patterns

### 4.1 API Gateway Architecture

**Structure**:
```
Clients
   │
   ▼
┌─────────────────┐
│  API Gateway    │
│  - Routing      │
│  - Auth         │
│  - Rate Limit   │
│  - Caching      │
└────────┬────────┘
         │
    ┌────┼────┐
    ▼    ▼    ▼
Service1 Service2 Service3
```

### 4.2 API Gateway Implementation

```java
@SpringBootApplication
@EnableZuulProxy
public class ApiGatewayApplication {
    public static void main(String[] args) {
        SpringApplication.run(ApiGatewayApplication.class, args);
    }
}

// Routing Configuration
@Configuration
class GatewayConfig {
    
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes()
            .route("user-service", r -> r
                .path("/api/users/**")
                .uri("lb://user-service"))
            .route("order-service", r -> r
                .path("/api/orders/**")
                .uri("lb://order-service"))
            .route("product-service", r -> r
                .path("/api/products/**")
                .uri("lb://product-service"))
            .build();
    }
}

// Authentication Filter
@Component
class AuthenticationFilter extends ZuulFilter {
    
    @Override
    public String filterType() {
        return "pre";
    }
    
    @Override
    public int filterOrder() {
        return 1;
    }
    
    @Override
    public boolean shouldFilter() {
        return true;
    }
    
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        
        String token = request.getHeader("Authorization");
        
        if (token == null || !isValidToken(token)) {
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            ctx.setResponseBody("Unauthorized");
        }
        
        return null;
    }
    
    private boolean isValidToken(String token) {
        // Validate JWT token
        return true;
    }
}

// Rate Limiting Filter
@Component
class RateLimitFilter extends ZuulFilter {
    
    private final RateLimiter rateLimiter = RateLimiter.create(100.0); // 100 req/s
    
    @Override
    public String filterType() {
        return "pre";
    }
    
    @Override
    public int filterOrder() {
        return 2;
    }
    
    @Override
    public boolean shouldFilter() {
        return true;
    }
    
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        
        if (!rateLimiter.tryAcquire()) {
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(429);
            ctx.setResponseBody("Rate limit exceeded");
        }
        
        return null;
    }
}
```

### 4.3 API Gateway Patterns

**Pattern 1: Backend for Frontend (BFF)**
```java
/**
 * BFF Pattern:
 * Different API gateways for different clients
 * 
 * Mobile BFF:
 *   - Optimized for mobile
 *   - Reduced payload
 *   - Mobile-specific endpoints
 * 
 * Web BFF:
 *   - Full features
 *   - Web-optimized responses
 */

@RestController
@RequestMapping("/mobile/api")
class MobileBFFController {
    
    @GetMapping("/users/{id}")
    public MobileUserResponse getUser(@PathVariable String id) {
        // Fetch user
        User user = userService.getUser(id);
        
        // Transform for mobile (reduced data)
        return new MobileUserResponse(
            user.getId(),
            user.getName(),
            user.getEmail()
            // Exclude unnecessary fields
        );
    }
}

@RestController
@RequestMapping("/web/api")
class WebBFFController {
    
    @GetMapping("/users/{id}")
    public WebUserResponse getUser(@PathVariable String id) {
        // Fetch user with all details
        User user = userService.getUser(id);
        
        // Full response for web
        return new WebUserResponse(user); // All fields
    }
}
```

**Pattern 2: API Aggregation**
```java
@RestController
@RequestMapping("/api/dashboard")
class DashboardController {
    
    @Autowired
    private UserServiceClient userServiceClient;
    
    @Autowired
    private OrderServiceClient orderServiceClient;
    
    @Autowired
    private ProductServiceClient productServiceClient;
    
    @GetMapping("/{userId}")
    public DashboardData getDashboard(@PathVariable String userId) {
        // Aggregate data from multiple services
        CompletableFuture<User> userFuture = CompletableFuture.supplyAsync(
            () -> userServiceClient.getUser(userId));
        
        CompletableFuture<List<Order>> ordersFuture = CompletableFuture.supplyAsync(
            () -> orderServiceClient.getUserOrders(userId));
        
        CompletableFuture<List<Product>> productsFuture = CompletableFuture.supplyAsync(
            () -> productServiceClient.getRecommendedProducts(userId));
        
        // Wait for all to complete
        CompletableFuture.allOf(userFuture, ordersFuture, productsFuture).join();
        
        return new DashboardData(
            userFuture.join(),
            ordersFuture.join(),
            productsFuture.join()
        );
    }
}
```

---

## 5. Practical Examples

### 5.1 Scalable E-Commerce Platform

**Complete Architecture**:
```java
/**
 * E-Commerce Platform Architecture:
 * 
 * Frontend:
 *   - CDN (CloudFront)
 *   - Browser caching
 * 
 * API Gateway:
 *   - Authentication
 *   - Rate limiting
 *   - Request routing
 *   - Response caching
 * 
 * Application Services:
 *   - User Service (stateless)
 *   - Product Service (with cache)
 *   - Order Service (with queue)
 *   - Payment Service
 *   - Inventory Service
 * 
 * Data Layer:
 *   - Primary DB (writes)
 *   - Read Replicas (reads)
 *   - Redis Cache
 *   - Elasticsearch (search)
 * 
 * Message Queue:
 *   - Order processing
 *   - Email notifications
 *   - Inventory updates
 */
```

**Order Processing Flow**:
```java
// 1. API Gateway receives request
@RestController
class OrderController {
    
    @Autowired
    private OrderService orderService;
    
    @PostMapping("/orders")
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        // Validate request
        // Authenticate user
        // Rate limit check
        
        Order order = orderService.createOrder(request);
        return ResponseEntity.ok(order);
    }
}

// 2. Order Service processes order
@Service
class OrderService {
    
    @Autowired
    private OrderRepository orderRepository;
    
    @Autowired
    private KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    @Autowired
    private InventoryServiceClient inventoryService;
    
    public Order createOrder(OrderRequest request) {
        // Check inventory (synchronous)
        boolean available = inventoryService.checkAvailability(request.getItems());
        if (!available) {
            throw new InsufficientInventoryException();
        }
        
        // Create order
        Order order = new Order(request);
        order = orderRepository.save(order);
        
        // Publish event (asynchronous)
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        kafkaTemplate.send("order-events", event);
        
        return order;
    }
}

// 3. Event Consumers process asynchronously
@Component
class OrderEventProcessor {
    
    @KafkaListener(topics = "order-events", groupId = "payment-processor")
    public void processPayment(OrderCreatedEvent event) {
        paymentService.processPayment(event.getOrderId());
    }
    
    @KafkaListener(topics = "order-events", groupId = "inventory-processor")
    public void updateInventory(OrderCreatedEvent event) {
        inventoryService.reserveItems(event.getOrderId(), event.getItems());
    }
    
    @KafkaListener(topics = "order-events", groupId = "notification-processor")
    public void sendNotification(OrderCreatedEvent event) {
        notificationService.sendOrderConfirmation(event);
    }
}
```

---

## Summary: Part 3

### Key Concepts

1. **Message Queues**: Decouple services, handle traffic spikes
2. **Event Streaming**: Real-time event processing with Kafka
3. **Event Sourcing**: Store state as sequence of events
4. **CQRS**: Separate read and write models
5. **API Gateway**: Single entry point, routing, auth, rate limiting

### Next Steps

**Part 4** will cover:
- CDN and Edge Computing
- Monitoring and Observability
- Performance Optimization
- Disaster Recovery

---

**Master message queues and microservices communication for scalable systems!**

