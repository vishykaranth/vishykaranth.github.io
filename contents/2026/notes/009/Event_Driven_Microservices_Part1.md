# Building Event-Driven Microservices in Java - Step by Step

## Part 1: Fundamentals and Architecture

---

## Table of Contents

1. [Introduction to Event-Driven Architecture](#1-introduction-to-event-driven-architecture)
2. [Core Concepts](#2-core-concepts)
3. [Event-Driven vs Request-Response](#3-event-driven-vs-request-response)
4. [Messaging Patterns](#4-messaging-patterns)
5. [Technology Stack](#5-technology-stack)
6. [System Architecture Overview](#6-system-architecture-overview)

---

## 1. Introduction to Event-Driven Architecture

### 1.1 What is Event-Driven Architecture?

Event-Driven Architecture (EDA) is a software architecture pattern that promotes the production, detection, consumption, and reaction to events. In this pattern, services communicate through events rather than direct API calls.

### 1.2 Key Characteristics

- **Decoupled Services**: Services don't need to know about each other
- **Asynchronous Communication**: Non-blocking message passing
- **Scalability**: Services can scale independently
- **Resilience**: Failure in one service doesn't cascade
- **Eventual Consistency**: Data consistency is achieved over time

### 1.3 Benefits

1. **Loose Coupling**: Services are independent
2. **Scalability**: Horizontal scaling is easier
3. **Flexibility**: Easy to add new services
4. **Resilience**: Isolated failures
5. **Real-time Processing**: Immediate event handling

### 1.4 Challenges

1. **Eventual Consistency**: Data may be temporarily inconsistent
2. **Complexity**: Debugging distributed systems is harder
3. **Event Ordering**: Maintaining order can be challenging
4. **Testing**: More complex than synchronous systems
5. **Monitoring**: Need comprehensive observability

---

## 2. Core Concepts

### 2.1 Events

An event is a significant occurrence or state change that has happened in the system.

```java
// Example: Order Created Event
public class OrderCreatedEvent {
    private String orderId;
    private String customerId;
    private LocalDateTime timestamp;
    private List<OrderItem> items;
    private BigDecimal totalAmount;
    
    // Constructors, getters, setters
    public OrderCreatedEvent() {}
    
    public OrderCreatedEvent(String orderId, String customerId, 
                           List<OrderItem> items, BigDecimal totalAmount) {
        this.orderId = orderId;
        this.customerId = customerId;
        this.timestamp = LocalDateTime.now();
        this.items = items;
        this.totalAmount = totalAmount;
    }
    
    // Getters and setters
    public String getOrderId() { return orderId; }
    public void setOrderId(String orderId) { this.orderId = orderId; }
    
    public String getCustomerId() { return customerId; }
    public void setCustomerId(String customerId) { this.customerId = customerId; }
    
    public LocalDateTime getTimestamp() { return timestamp; }
    public void setTimestamp(LocalDateTime timestamp) { this.timestamp = timestamp; }
    
    public List<OrderItem> getItems() { return items; }
    public void setItems(List<OrderItem> items) { this.items = items; }
    
    public BigDecimal getTotalAmount() { return totalAmount; }
    public void setTotalAmount(BigDecimal totalAmount) { this.totalAmount = totalAmount; }
}
```

### 2.2 Event Producers (Publishers)

Services that generate and publish events.

```java
// Example: Order Service publishes OrderCreatedEvent
@Service
public class OrderService {
    
    private final EventPublisher eventPublisher;
    
    public OrderService(EventPublisher eventPublisher) {
        this.eventPublisher = eventPublisher;
    }
    
    public Order createOrder(CreateOrderRequest request) {
        // Business logic to create order
        Order order = new Order();
        order.setOrderId(UUID.randomUUID().toString());
        order.setCustomerId(request.getCustomerId());
        order.setItems(request.getItems());
        order.setTotalAmount(calculateTotal(request.getItems()));
        order.setStatus(OrderStatus.CREATED);
        
        // Save to database
        orderRepository.save(order);
        
        // Publish event
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getOrderId(),
            order.getCustomerId(),
            order.getItems(),
            order.getTotalAmount()
        );
        eventPublisher.publish("order.created", event);
        
        return order;
    }
}
```

### 2.3 Event Consumers (Subscribers)

Services that listen for and react to events.

```java
// Example: Inventory Service consumes OrderCreatedEvent
@Service
public class InventoryService {
    
    @EventListener("order.created")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Update inventory
        for (OrderItem item : event.getItems()) {
            inventoryRepository.decreaseStock(
                item.getProductId(),
                item.getQuantity()
            );
        }
        
        // Publish inventory updated event
        InventoryUpdatedEvent inventoryEvent = new InventoryUpdatedEvent(
            event.getOrderId(),
            event.getItems()
        );
        eventPublisher.publish("inventory.updated", inventoryEvent);
    }
}
```

### 2.4 Event Broker/Message Queue

Middleware that routes events from producers to consumers.

**Popular Options:**
- **RabbitMQ**: Easy to use, good for complex routing
- **Apache Kafka**: High throughput, event streaming
- **Apache Pulsar**: Multi-tenancy, geo-replication
- **Amazon SQS/SNS**: Managed service on AWS
- **Azure Service Bus**: Managed service on Azure

---

## 3. Event-Driven vs Request-Response

### 3.1 Request-Response Pattern

```
┌─────────┐         HTTP/REST         ┌─────────┐
│ Client  │ ────────────────────────► │ Service │
│         │                            │    A    │
└─────────┘                            └────┬────┘
                                            │
                                            │ HTTP/REST
                                            ▼
                                      ┌─────────┐
                                      │ Service │
                                      │    B    │
                                      └─────────┘
```

**Characteristics:**
- Synchronous
- Tight coupling
- Immediate response
- Blocking calls
- Direct dependencies

### 3.2 Event-Driven Pattern

```
┌─────────┐                            ┌─────────┐
│ Service │  ──► Event ──►  ┌────────┐  │ Service │
│    A    │                 │ Broker │  │    B    │
└─────────┘                 └────┬───┘  └─────────┘
                                  │
                                  │ Event
                                  ▼
                            ┌─────────┐
                            │ Service │
                            │    C    │
                            └─────────┘
```

**Characteristics:**
- Asynchronous
- Loose coupling
- Eventual consistency
- Non-blocking
- Indirect dependencies

### 3.3 Comparison Table

| Aspect | Request-Response | Event-Driven |
|--------|------------------|--------------|
| **Coupling** | Tight | Loose |
| **Communication** | Synchronous | Asynchronous |
| **Response Time** | Immediate | Eventual |
| **Scalability** | Limited | High |
| **Failure Handling** | Cascading | Isolated |
| **Complexity** | Lower | Higher |
| **Use Case** | Real-time queries | Background processing |

---

## 4. Messaging Patterns

### 4.1 Point-to-Point (Queue)

One producer, one consumer. Each message is consumed by exactly one consumer.

```
Producer ──► [Queue] ──► Consumer
```

**Use Cases:**
- Task distribution
- Load balancing
- Work queues

### 4.2 Publish-Subscribe (Topic/Exchange)

One producer, multiple consumers. Each message is delivered to all subscribers.

```
Producer ──► [Topic/Exchange] ──► Consumer 1
                              └─► Consumer 2
                              └─► Consumer 3
```

**Use Cases:**
- Event notifications
- Broadcast messages
- Multiple service reactions

### 4.3 Request-Reply

Producer sends a message and waits for a reply.

```
Producer ──► [Request Queue] ──► Consumer
Producer ◄── [Reply Queue] ◄─── Consumer
```

**Use Cases:**
- RPC over messaging
- Query operations
- Synchronous-like behavior

### 4.4 Event Sourcing

Store all changes as a sequence of events.

```
Event Store: [Event1, Event2, Event3, ...]
Current State = Replay all events
```

**Use Cases:**
- Audit trail
- Time travel
- Complex state reconstruction

### 4.5 Saga Pattern

Orchestrate distributed transactions using events.

```
Order Service ──► OrderCreated
                    │
                    ├─► Payment Service ──► PaymentProcessed
                    │
                    └─► Inventory Service ──► InventoryReserved
```

**Use Cases:**
- Distributed transactions
- Long-running processes
- Compensation logic

---

## 5. Technology Stack

### 5.1 Core Technologies

#### Spring Boot
- Framework for building microservices
- Spring Cloud for distributed systems
- Spring AMQP for RabbitMQ integration
- Spring Kafka for Kafka integration

#### Message Brokers

**RabbitMQ:**
- Easy setup and management
- Good for complex routing
- Supports multiple protocols
- Management UI available

**Apache Kafka:**
- High throughput
- Event streaming
- Log-based storage
- Horizontal scaling

### 5.2 Project Structure

```
event-driven-microservices/
├── order-service/
│   ├── src/main/java/
│   │   └── com/example/order/
│   │       ├── OrderApplication.java
│   │       ├── controller/
│   │       ├── service/
│   │       ├── repository/
│   │       ├── model/
│   │       └── event/
│   ├── src/main/resources/
│   │   └── application.yml
│   └── pom.xml
├── payment-service/
│   └── (similar structure)
├── inventory-service/
│   └── (similar structure)
├── notification-service/
│   └── (similar structure)
└── common/
    └── (shared models and utilities)
```

### 5.3 Dependencies (Maven)

```xml
<!-- Spring Boot Starter -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- Spring AMQP (RabbitMQ) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>

<!-- Spring Kafka (if using Kafka) -->
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>

<!-- Lombok (optional, for reducing boilerplate) -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <optional>true</optional>
</dependency>
```

---

## 6. System Architecture Overview

### 6.1 E-Commerce Example System

We'll build an e-commerce system with the following services:

```
┌─────────────────────────────────────────────────────────────┐
│                    E-Commerce System                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐    ┌──────────────┐   ┌──────────────┐ │
│  │   Order      │    │   Payment    │   │  Inventory   │ │
│  │   Service    │    │   Service    │   │   Service    │ │
│  └──────┬───────┘    └──────┬───────┘   └──────┬───────┘ │
│         │                    │                   │         │
│         └────────────────────┼───────────────────┘         │
│                              │                             │
│                    ┌─────────▼─────────┐                  │
│                    │   Message Broker  │                  │
│                    │   (RabbitMQ)      │                  │
│                    └─────────┬─────────┘                  │
│                              │                             │
│         ┌────────────────────┼───────────────────┐         │
│         │                    │                   │         │
│  ┌──────▼───────┐   ┌────────▼──────┐  ┌──────▼───────┐ │
│  │ Notification │   │   Shipping    │  │   Analytics  │ │
│  │   Service    │   │   Service     │  │   Service     │ │
│  └──────────────┘   └───────────────┘  └───────────────┘ │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

### 6.2 Event Flow Example

**Scenario: Customer places an order**

1. **Order Service** receives order request
2. **Order Service** creates order and publishes `OrderCreatedEvent`
3. **Payment Service** consumes event, processes payment, publishes `PaymentProcessedEvent`
4. **Inventory Service** consumes event, reserves items, publishes `InventoryReservedEvent`
5. **Shipping Service** consumes events, creates shipment
6. **Notification Service** consumes events, sends notifications
7. **Analytics Service** consumes events, updates analytics

### 6.3 Event Types

```java
// Order Events
- OrderCreatedEvent
- OrderCancelledEvent
- OrderShippedEvent
- OrderDeliveredEvent

// Payment Events
- PaymentProcessedEvent
- PaymentFailedEvent
- PaymentRefundedEvent

// Inventory Events
- InventoryReservedEvent
- InventoryReleasedEvent
- InventoryLowStockEvent

// Shipping Events
- ShipmentCreatedEvent
- ShipmentDeliveredEvent
```

### 6.4 Service Responsibilities

**Order Service:**
- Create orders
- Manage order lifecycle
- Publish order events

**Payment Service:**
- Process payments
- Handle refunds
- Publish payment events

**Inventory Service:**
- Manage stock
- Reserve/release inventory
- Publish inventory events

**Shipping Service:**
- Create shipments
- Track deliveries
- Publish shipping events

**Notification Service:**
- Send emails
- Send SMS
- Push notifications

**Analytics Service:**
- Track events
- Generate reports
- Business intelligence

---

## Summary of Part 1

**Key Takeaways:**
1. **Event-Driven Architecture** enables loose coupling and scalability
2. **Events** represent state changes in the system
3. **Producers** publish events, **Consumers** react to events
4. **Message Broker** routes events between services
5. **Messaging Patterns** solve different communication needs
6. **Technology Stack** includes Spring Boot and message brokers (RabbitMQ/Kafka)

**Next Steps:**
- Part 2 will cover setting up the infrastructure (RabbitMQ, Spring Boot configuration)
- Part 3 will implement microservices with event publishing and consuming
- Part 4 will cover advanced topics (Saga pattern, event sourcing, monitoring)

**Prerequisites for Next Part:**
- Java 11 or higher
- Maven 3.6+
- Docker (for RabbitMQ)
- IDE (IntelliJ IDEA, Eclipse, or VS Code)

