# Building Event-Driven Microservices in Java - Step by Step

## Part 4: Advanced Patterns, Monitoring, and Best Practices

---

## Table of Contents

1. [Saga Pattern Implementation](#1-saga-pattern-implementation)
2. [Event Sourcing](#2-event-sourcing)
3. [Error Handling and Dead Letter Queues](#3-error-handling-and-dead-letter-queues)
4. [Monitoring and Observability](#4-monitoring-and-observability)
5. [Idempotency and Event Deduplication](#5-idempotency-and-event-deduplication)
6. [Performance Optimization](#6-performance-optimization)
7. [Best Practices Summary](#7-best-practices-summary)

---

## 1. Saga Pattern Implementation

### 1.1 What is Saga Pattern?

The Saga pattern manages distributed transactions by breaking them into a series of local transactions, each with a compensating action if the saga needs to be rolled back.

### 1.2 Saga Orchestrator

```java
package com.example.order.saga;

import com.example.common.event.*;
import com.example.order.model.Order;
import com.example.order.model.OrderStatus;
import com.example.order.repository.OrderRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

@Component
public class OrderSagaOrchestrator {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderSagaOrchestrator.class);
    
    private final OrderRepository orderRepository;
    private final Map<String, SagaState> sagaStates = new ConcurrentHashMap<>();
    
    @Autowired
    public OrderSagaOrchestrator(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }
    
    public void startSaga(Order order) {
        SagaState state = new SagaState(order.getOrderId());
        state.setOrderCreated(true);
        sagaStates.put(order.getOrderId(), state);
        logger.info("Saga started for order: {}", order.getOrderId());
    }
    
    @RabbitListener(queues = "payment.processed.queue")
    public void handlePaymentProcessed(PaymentProcessedEvent event) {
        SagaState state = sagaStates.get(event.getOrderId());
        if (state != null) {
            state.setPaymentProcessed(true);
            state.setPaymentStatus(event.getStatus());
            
            if (event.getStatus() == PaymentStatus.SUCCESS) {
                checkSagaCompletion(event.getOrderId());
            } else {
                compensateOrder(event.getOrderId(), "Payment failed");
            }
        }
    }
    
    @RabbitListener(queues = "inventory.reserved.queue")
    public void handleInventoryReserved(InventoryReservedEvent event) {
        SagaState state = sagaStates.get(event.getOrderId());
        if (state != null) {
            state.setInventoryReserved(event.isSuccess());
            
            if (event.isSuccess()) {
                checkSagaCompletion(event.getOrderId());
            } else {
                compensateOrder(event.getOrderId(), "Inventory reservation failed");
            }
        }
    }
    
    private void checkSagaCompletion(String orderId) {
        SagaState state = sagaStates.get(orderId);
        if (state != null && 
            state.isOrderCreated() && 
            state.isPaymentProcessed() && 
            state.getPaymentStatus() == PaymentStatus.SUCCESS &&
            state.isInventoryReserved()) {
            
            // All steps completed successfully
            Order order = orderRepository.findByOrderId(orderId)
                .orElseThrow(() -> new RuntimeException("Order not found: " + orderId));
            order.setStatus(OrderStatus.PAID);
            orderRepository.save(order);
            
            logger.info("Saga completed successfully for order: {}", orderId);
            sagaStates.remove(orderId);
        }
    }
    
    private void compensateOrder(String orderId, String reason) {
        Order order = orderRepository.findByOrderId(orderId)
            .orElseThrow(() -> new RuntimeException("Order not found: " + orderId));
        
        order.setStatus(OrderStatus.CANCELLED);
        orderRepository.save(order);
        
        // Publish compensation events
        // Release inventory, refund payment, etc.
        
        logger.warn("Saga compensated for order: {} - Reason: {}", orderId, reason);
        sagaStates.remove(orderId);
    }
}

class SagaState {
    private String orderId;
    private boolean orderCreated;
    private boolean paymentProcessed;
    private PaymentStatus paymentStatus;
    private boolean inventoryReserved;
    
    public SagaState(String orderId) {
        this.orderId = orderId;
        this.orderCreated = false;
        this.paymentProcessed = false;
        this.inventoryReserved = false;
    }
    
    // Getters and setters
    public String getOrderId() {
        return orderId;
    }
    
    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }
    
    public boolean isOrderCreated() {
        return orderCreated;
    }
    
    public void setOrderCreated(boolean orderCreated) {
        this.orderCreated = orderCreated;
    }
    
    public boolean isPaymentProcessed() {
        return paymentProcessed;
    }
    
    public void setPaymentProcessed(boolean paymentProcessed) {
        this.paymentProcessed = paymentProcessed;
    }
    
    public PaymentStatus getPaymentStatus() {
        return paymentStatus;
    }
    
    public void setPaymentStatus(PaymentStatus paymentStatus) {
        this.paymentStatus = paymentStatus;
    }
    
    public boolean isInventoryReserved() {
        return inventoryReserved;
    }
    
    public void setInventoryReserved(boolean inventoryReserved) {
        this.inventoryReserved = inventoryReserved;
    }
}
```

### 1.3 Compensation Events

```java
package com.example.common.event;

public class OrderCancelledEvent extends BaseEvent {
    private String orderId;
    private String reason;
    
    public OrderCancelledEvent() {
        super("OrderCancelledEvent");
    }
    
    public OrderCancelledEvent(String orderId, String reason) {
        super("OrderCancelledEvent");
        this.orderId = orderId;
        this.reason = reason;
    }
    
    // Getters and setters
    public String getOrderId() {
        return orderId;
    }
    
    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }
    
    public String getReason() {
        return reason;
    }
    
    public void setReason(String reason) {
        this.reason = reason;
    }
}

public class InventoryReleasedEvent extends BaseEvent {
    private String orderId;
    private List<InventoryItem> releasedItems;
    
    public InventoryReleasedEvent() {
        super("InventoryReleasedEvent");
    }
    
    public InventoryReleasedEvent(String orderId, List<InventoryItem> releasedItems) {
        super("InventoryReleasedEvent");
        this.orderId = orderId;
        this.releasedItems = releasedItems;
    }
    
    // Getters and setters
    public String getOrderId() {
        return orderId;
    }
    
    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }
    
    public List<InventoryItem> getReleasedItems() {
        return releasedItems;
    }
    
    public void setReleasedItems(List<InventoryItem> releasedItems) {
        this.releasedItems = releasedItems;
    }
}
```

---

## 2. Event Sourcing

### 2.1 What is Event Sourcing?

Event Sourcing stores all changes to application state as a sequence of events. Instead of storing current state, we store events and reconstruct state by replaying them.

### 2.2 Event Store

```java
package com.example.common.eventsourcing;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "event_store")
public class EventStore {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String aggregateId;
    private String eventType;
    private String eventData;
    private Long version;
    private LocalDateTime timestamp;
    
    @PrePersist
    protected void onCreate() {
        timestamp = LocalDateTime.now();
    }
    
    // Getters and setters
    public Long getId() {
        return id;
    }
    
    public void setId(Long id) {
        this.id = id;
    }
    
    public String getAggregateId() {
        return aggregateId;
    }
    
    public void setAggregateId(String aggregateId) {
        this.aggregateId = aggregateId;
    }
    
    public String getEventType() {
        return eventType;
    }
    
    public void setEventType(String eventType) {
        this.eventType = eventType;
    }
    
    public String getEventData() {
        return eventData;
    }
    
    public void setEventData(String eventData) {
        this.eventData = eventData;
    }
    
    public Long getVersion() {
        return version;
    }
    
    public void setVersion(Long version) {
        this.version = version;
    }
    
    public LocalDateTime getTimestamp() {
        return timestamp;
    }
    
    public void setTimestamp(LocalDateTime timestamp) {
        this.timestamp = timestamp;
    }
}
```

### 2.3 Event Store Repository

```java
package com.example.common.eventsourcing;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface EventStoreRepository extends JpaRepository<EventStore, Long> {
    List<EventStore> findByAggregateIdOrderByVersionAsc(String aggregateId);
    List<EventStore> findByAggregateIdAndVersionGreaterThan(String aggregateId, Long version);
}
```

### 2.4 Event Store Service

```java
package com.example.common.eventsourcing;

import com.example.common.event.BaseEvent;
import com.fasterxml.jackson.core.JsonProcessingException;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;
import java.util.stream.Collectors;

@Service
public class EventStoreService {
    
    private static final Logger logger = LoggerFactory.getLogger(EventStoreService.class);
    
    private final EventStoreRepository eventStoreRepository;
    private final ObjectMapper objectMapper;
    
    @Autowired
    public EventStoreService(EventStoreRepository eventStoreRepository, ObjectMapper objectMapper) {
        this.eventStoreRepository = eventStoreRepository;
        this.objectMapper = objectMapper;
    }
    
    @Transactional
    public void saveEvent(String aggregateId, BaseEvent event) {
        try {
            EventStore eventStore = new EventStore();
            eventStore.setAggregateId(aggregateId);
            eventStore.setEventType(event.getEventType());
            eventStore.setEventData(objectMapper.writeValueAsString(event));
            
            // Get next version
            Long currentVersion = eventStoreRepository
                .findByAggregateIdOrderByVersionAsc(aggregateId)
                .stream()
                .mapToLong(EventStore::getVersion)
                .max()
                .orElse(0L);
            
            eventStore.setVersion(currentVersion + 1);
            eventStoreRepository.save(eventStore);
            
            logger.info("Event stored: {} for aggregate: {} version: {}", 
                event.getEventType(), aggregateId, eventStore.getVersion());
        } catch (JsonProcessingException e) {
            logger.error("Error serializing event", e);
            throw new RuntimeException("Failed to save event", e);
        }
    }
    
    public List<BaseEvent> getEvents(String aggregateId) {
        List<EventStore> events = eventStoreRepository
            .findByAggregateIdOrderByVersionAsc(aggregateId);
        
        return events.stream()
            .map(event -> {
                try {
                    return (BaseEvent) objectMapper.readValue(
                        event.getEventData(), 
                        Class.forName("com.example.common.event." + event.getEventType())
                    );
                } catch (Exception e) {
                    logger.error("Error deserializing event", e);
                    throw new RuntimeException("Failed to load event", e);
                }
            })
            .collect(Collectors.toList());
    }
    
    public List<BaseEvent> getEventsSince(String aggregateId, Long version) {
        List<EventStore> events = eventStoreRepository
            .findByAggregateIdAndVersionGreaterThan(aggregateId, version);
        
        return events.stream()
            .map(event -> {
                try {
                    return (BaseEvent) objectMapper.readValue(
                        event.getEventData(), 
                        Class.forName("com.example.common.event." + event.getEventType())
                    );
                } catch (Exception e) {
                    logger.error("Error deserializing event", e);
                    throw new RuntimeException("Failed to load event", e);
                }
            })
            .collect(Collectors.toList());
    }
}
```

### 2.5 Using Event Store in Order Service

```java
package com.example.order.service;

import com.example.common.eventsourcing.EventStoreService;
import com.example.common.event.OrderCreatedEvent;
import com.example.order.model.Order;
import com.example.order.repository.OrderRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderServiceWithEventSourcing {
    
    private final OrderRepository orderRepository;
    private final EventStoreService eventStoreService;
    
    @Autowired
    public OrderServiceWithEventSourcing(OrderRepository orderRepository, 
                                         EventStoreService eventStoreService) {
        this.orderRepository = orderRepository;
        this.eventStoreService = eventStoreService;
    }
    
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        // Create order
        Order order = new Order();
        order.setOrderId(UUID.randomUUID().toString());
        // ... set other fields
        
        // Save order
        order = orderRepository.save(order);
        
        // Store event
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getOrderId(),
            order.getCustomerId(),
            convertToEventItems(order.getItems()),
            order.getTotalAmount()
        );
        eventStoreService.saveEvent(order.getOrderId(), event);
        
        return order;
    }
    
    public Order reconstructOrder(String orderId) {
        // Reconstruct order from events
        List<BaseEvent> events = eventStoreService.getEvents(orderId);
        Order order = new Order();
        
        for (BaseEvent event : events) {
            if (event instanceof OrderCreatedEvent) {
                OrderCreatedEvent createdEvent = (OrderCreatedEvent) event;
                order.setOrderId(createdEvent.getOrderId());
                order.setCustomerId(createdEvent.getCustomerId());
                order.setTotalAmount(createdEvent.getTotalAmount());
                // ... reconstruct other fields
            }
            // Handle other event types
        }
        
        return order;
    }
}
```

---

## 3. Error Handling and Dead Letter Queues

### 3.1 Dead Letter Queue Configuration

```java
package com.example.common.config;

import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.ConditionalRejectingErrorHandler;
import org.springframework.amqp.rabbit.listener.FatalExceptionStrategy;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class DeadLetterQueueConfig {
    
    // Dead Letter Exchange
    @Bean
    public DirectExchange deadLetterExchange() {
        return new DirectExchange("dlx", true, false);
    }
    
    // Dead Letter Queue
    @Bean
    public Queue deadLetterQueue() {
        return QueueBuilder.durable("dlq").build();
    }
    
    // Binding
    @Bean
    public Binding deadLetterBinding() {
        return BindingBuilder
            .bind(deadLetterQueue())
            .to(deadLetterExchange())
            .with("dlq");
    }
    
    // Configure queues with DLX
    @Bean
    public Queue orderCreatedQueueWithDLQ() {
        return QueueBuilder.durable("order.created.queue")
            .withArgument("x-dead-letter-exchange", "dlx")
            .withArgument("x-dead-letter-routing-key", "dlq")
            .withArgument("x-message-ttl", 60000) // 60 seconds TTL
            .build();
    }
    
    // Error Handler
    @Bean
    public ConditionalRejectingErrorHandler errorHandler() {
        return new ConditionalRejectingErrorHandler(new FatalExceptionStrategy() {
            @Override
            public boolean isFatal(Throwable t) {
                // Don't retry for certain exceptions
                return t instanceof IllegalArgumentException ||
                       t instanceof NullPointerException;
            }
        });
    }
    
    // Listener Container Factory with Error Handler
    @Bean
    public SimpleRabbitListenerContainerFactory rabbitListenerContainerFactory(
            ConnectionFactory connectionFactory) {
        SimpleRabbitListenerContainerFactory factory = new SimpleRabbitListenerContainerFactory();
        factory.setConnectionFactory(connectionFactory);
        factory.setErrorHandler(errorHandler());
        factory.setDefaultRequeueRejected(false); // Don't requeue failed messages
        return factory;
    }
}
```

### 3.2 Retry Mechanism

```java
package com.example.common.config;

import org.springframework.amqp.rabbit.config.SimpleRabbitListenerContainerFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.rabbit.retry.MessageRecoverer;
import org.springframework.amqp.rabbit.retry.RejectAndDontRequeueRecoverer;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.retry.backoff.ExponentialBackOffPolicy;
import org.springframework.retry.policy.SimpleRetryPolicy;
import org.springframework.retry.support.RetryTemplate;

@Configuration
public class RetryConfig {
    
    @Bean
    public RetryTemplate retryTemplate() {
        RetryTemplate retryTemplate = new RetryTemplate();
        
        // Retry policy: max 3 attempts
        SimpleRetryPolicy retryPolicy = new SimpleRetryPolicy();
        retryPolicy.setMaxAttempts(3);
        retryTemplate.setRetryPolicy(retryPolicy);
        
        // Backoff policy: exponential backoff
        ExponentialBackOffPolicy backOffPolicy = new ExponentialBackOffPolicy();
        backOffPolicy.setInitialInterval(1000); // 1 second
        backOffPolicy.setMultiplier(2.0); // Double each time
        backOffPolicy.setMaxInterval(10000); // Max 10 seconds
        retryTemplate.setBackOffPolicy(backOffPolicy);
        
        return retryTemplate;
    }
    
    @Bean
    public MessageRecoverer messageRecoverer() {
        // Send to DLQ after retries exhausted
        return new RejectAndDontRequeueRecoverer();
    }
}
```

### 3.3 Error Handling in Event Listeners

```java
package com.example.payment.listener;

import com.example.common.event.OrderCreatedEvent;
import com.example.payment.service.PaymentService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.retry.annotation.Backoff;
import org.springframework.retry.annotation.Retryable;
import org.springframework.stereotype.Component;

@Component
public class PaymentEventListenerWithRetry {
    
    private static final Logger logger = LoggerFactory.getLogger(PaymentEventListenerWithRetry.class);
    
    private final PaymentService paymentService;
    
    @Autowired
    public PaymentEventListenerWithRetry(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
    
    @RabbitListener(queues = "order.created.queue")
    @Retryable(
        value = {Exception.class},
        maxAttempts = 3,
        backoff = @Backoff(delay = 1000, multiplier = 2)
    )
    public void handleOrderCreated(OrderCreatedEvent event) {
        try {
            logger.info("Processing payment for order: {}", event.getOrderId());
            paymentService.processPayment(event);
        } catch (Exception e) {
            logger.error("Error processing payment for order: {}", event.getOrderId(), e);
            // Message will be retried or sent to DLQ
            throw e;
        }
    }
}
```

### 3.4 DLQ Handler

```java
package com.example.common.listener;

import com.example.common.event.BaseEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public class DeadLetterQueueHandler {
    
    private static final Logger logger = LoggerFactory.getLogger(DeadLetterQueueHandler.class);
    
    @RabbitListener(queues = "dlq")
    public void handleDeadLetterMessage(BaseEvent event) {
        logger.error("Received message in DLQ: EventType={}, EventId={}", 
            event.getEventType(), event.getEventId());
        
        // Log for manual investigation
        // Send alert to monitoring system
        // Store in database for analysis
        
        // Optionally, try to recover or notify administrators
        notifyAdministrators(event);
    }
    
    private void notifyAdministrators(BaseEvent event) {
        // Send notification to administrators
        logger.warn("Notifying administrators about failed event: {}", event.getEventId());
    }
}
```

---

## 4. Monitoring and Observability

### 4.1 Distributed Tracing

```java
package com.example.common.tracing;

import org.slf4j.MDC;
import org.springframework.amqp.core.Message;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.MessageConverter;
import org.springframework.stereotype.Component;

import java.util.UUID;

@Component
public class TracingRabbitTemplate extends RabbitTemplate {
    
    @Override
    public void convertAndSend(String exchange, String routingKey, Object object) {
        // Add trace ID to message headers
        String traceId = MDC.get("traceId");
        if (traceId == null) {
            traceId = UUID.randomUUID().toString();
            MDC.put("traceId", traceId);
        }
        
        super.convertAndSend(exchange, routingKey, object, message -> {
            message.getMessageProperties().setHeader("X-Trace-Id", traceId);
            return message;
        });
    }
}
```

### 4.2 Event Metrics

```java
package com.example.common.metrics;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.time.Duration;

@Component
public class EventMetrics {
    
    private final Counter eventsPublished;
    private final Counter eventsConsumed;
    private final Counter eventsFailed;
    private final Timer eventProcessingTime;
    
    public EventMetrics(MeterRegistry meterRegistry) {
        this.eventsPublished = Counter.builder("events.published")
            .description("Total events published")
            .register(meterRegistry);
        
        this.eventsConsumed = Counter.builder("events.consumed")
            .description("Total events consumed")
            .register(meterRegistry);
        
        this.eventsFailed = Counter.builder("events.failed")
            .description("Total events failed")
            .register(meterRegistry);
        
        this.eventProcessingTime = Timer.builder("events.processing.time")
            .description("Event processing time")
            .register(meterRegistry);
    }
    
    public void recordEventPublished(String eventType) {
        eventsPublished.increment("event.type", eventType);
    }
    
    public void recordEventConsumed(String eventType) {
        eventsConsumed.increment("event.type", eventType);
    }
    
    public void recordEventFailed(String eventType) {
        eventsFailed.increment("event.type", eventType);
    }
    
    public void recordProcessingTime(String eventType, Duration duration) {
        eventProcessingTime.record(duration);
    }
}
```

### 4.3 Health Checks

```java
package com.example.common.health;

import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.boot.actuate.health.Health;
import org.springframework.boot.actuate.health.HealthIndicator;
import org.springframework.stereotype.Component;

@Component
public class RabbitMQHealthIndicator implements HealthIndicator {
    
    private final RabbitTemplate rabbitTemplate;
    
    public RabbitMQHealthIndicator(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }
    
    @Override
    public Health health() {
        try {
            rabbitTemplate.execute(channel -> {
                channel.queueDeclarePassive("health.check");
                return null;
            });
            return Health.up()
                .withDetail("status", "RabbitMQ is available")
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

### 4.4 Logging Best Practices

```java
package com.example.common.logging;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.util.UUID;

@Component
public class EventLogger {
    
    private static final Logger logger = LoggerFactory.getLogger(EventLogger.class);
    
    @RabbitListener(queues = "order.created.queue")
    public void logEvent(BaseEvent event) {
        // Set trace ID in MDC
        String traceId = UUID.randomUUID().toString();
        MDC.put("traceId", traceId);
        MDC.put("eventId", event.getEventId());
        MDC.put("eventType", event.getEventType());
        
        try {
            logger.info("Processing event: {}", event.getEventType());
            // Process event
            logger.info("Event processed successfully");
        } catch (Exception e) {
            logger.error("Event processing failed", e);
            throw e;
        } finally {
            MDC.clear();
        }
    }
}
```

---

## 5. Idempotency and Event Deduplication

### 5.1 Event Deduplication

```java
package com.example.common.deduplication;

import com.example.common.event.BaseEvent;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

@Component
public class EventDeduplicator {
    
    private final Set<String> processedEvents = ConcurrentHashMap.newKeySet();
    
    @RabbitListener(queues = "order.created.queue")
    public void handleEvent(BaseEvent event) {
        String eventId = event.getEventId();
        
        if (processedEvents.contains(eventId)) {
            // Event already processed, skip
            return;
        }
        
        try {
            // Process event
            processEvent(event);
            
            // Mark as processed
            processedEvents.add(eventId);
        } catch (Exception e) {
            // Don't mark as processed if failed
            throw e;
        }
    }
    
    private void processEvent(BaseEvent event) {
        // Event processing logic
    }
}
```

### 5.2 Idempotent Event Handler

```java
package com.example.payment.service;

import com.example.common.event.OrderCreatedEvent;
import com.example.payment.model.Payment;
import com.example.payment.repository.PaymentRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class IdempotentPaymentService {
    
    private final PaymentRepository paymentRepository;
    
    @Autowired
    public IdempotentPaymentService(PaymentRepository paymentRepository) {
        this.paymentRepository = paymentRepository;
    }
    
    @Transactional
    public void processPayment(OrderCreatedEvent event) {
        // Check if payment already processed (idempotency check)
        Payment existingPayment = paymentRepository.findByOrderId(event.getOrderId())
            .orElse(null);
        
        if (existingPayment != null) {
            // Payment already processed, return
            return;
        }
        
        // Process payment
        Payment payment = new Payment();
        payment.setPaymentId(UUID.randomUUID().toString());
        payment.setOrderId(event.getOrderId());
        // ... set other fields
        
        paymentRepository.save(payment);
    }
}
```

---

## 6. Performance Optimization

### 6.1 Batch Processing

```java
package com.example.common.batch;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

import java.util.List;

@Component
public class BatchEventProcessor {
    
    @RabbitListener(queues = "order.created.queue", containerFactory = "batchContainerFactory")
    public void processBatch(List<OrderCreatedEvent> events) {
        // Process batch of events
        for (OrderCreatedEvent event : events) {
            // Process each event
        }
    }
}
```

### 6.2 Async Processing

```java
package com.example.common.async;

import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

@Component
public class AsyncEventProcessor {
    
    @RabbitListener(queues = "order.created.queue")
    @Async
    public void processAsync(OrderCreatedEvent event) {
        // Process event asynchronously
    }
}
```

### 6.3 Connection Pooling

```yaml
# application.yml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin
    virtual-host: /
    listener:
      simple:
        concurrency: 5
        max-concurrency: 10
        prefetch: 10
    template:
      retry:
        enabled: true
        initial-interval: 1000
        max-attempts: 3
        multiplier: 2
```

---

## 7. Best Practices Summary

### 7.1 Event Design

1. **Make events immutable**: Once published, events should not change
2. **Include all necessary data**: Events should be self-contained
3. **Version events**: Use versioning for event schema evolution
4. **Use meaningful names**: Event names should clearly describe what happened

### 7.2 Service Design

1. **Keep services independent**: Services should not depend on each other
2. **Handle failures gracefully**: Implement retry and compensation logic
3. **Ensure idempotency**: Event handlers should be idempotent
4. **Monitor everything**: Implement comprehensive monitoring

### 7.3 Error Handling

1. **Use Dead Letter Queues**: For messages that cannot be processed
2. **Implement retries**: With exponential backoff
3. **Log errors**: Comprehensive error logging
4. **Alert on failures**: Notify administrators of critical failures

### 7.4 Performance

1. **Use async processing**: Don't block on event processing
2. **Batch when possible**: Process multiple events together
3. **Optimize database queries**: Use appropriate indexes
4. **Monitor performance**: Track processing times

### 7.5 Security

1. **Encrypt sensitive data**: In events and queues
2. **Authenticate services**: Use service-to-service authentication
3. **Validate events**: Validate event data before processing
4. **Audit events**: Log all events for audit purposes

### 7.6 Testing

1. **Unit test event handlers**: Test business logic in isolation
2. **Integration tests**: Test event flow between services
3. **Contract tests**: Test event schemas
4. **Load tests**: Test system under load

---

## Complete Example: Order Processing with All Patterns

```java
package com.example.order.service;

import com.example.common.event.*;
import com.example.common.eventsourcing.EventStoreService;
import com.example.common.metrics.EventMetrics;
import com.example.order.model.Order;
import com.example.order.repository.OrderRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.Duration;
import java.time.Instant;

@Service
public class CompleteOrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(CompleteOrderService.class);
    
    private final OrderRepository orderRepository;
    private final EventPublisher eventPublisher;
    private final EventStoreService eventStoreService;
    private final EventMetrics eventMetrics;
    
    @Autowired
    public CompleteOrderService(OrderRepository orderRepository,
                               EventPublisher eventPublisher,
                               EventStoreService eventStoreService,
                               EventMetrics eventMetrics) {
        this.orderRepository = orderRepository;
        this.eventPublisher = eventPublisher;
        this.eventStoreService = eventStoreService;
        this.eventMetrics = eventMetrics;
    }
    
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        Instant start = Instant.now();
        
        try {
            // Create order
            Order order = new Order();
            order.setOrderId(UUID.randomUUID().toString());
            order.setCustomerId(request.getCustomerId());
            // ... set other fields
            
            order = orderRepository.save(order);
            
            // Create and store event
            OrderCreatedEvent event = new OrderCreatedEvent(
                order.getOrderId(),
                order.getCustomerId(),
                convertToEventItems(order.getItems()),
                order.getTotalAmount()
            );
            
            // Store in event store (Event Sourcing)
            eventStoreService.saveEvent(order.getOrderId(), event);
            
            // Publish event
            eventPublisher.publish("order.created", event);
            
            // Record metrics
            eventMetrics.recordEventPublished("OrderCreatedEvent");
            eventMetrics.recordProcessingTime("OrderCreatedEvent", 
                Duration.between(start, Instant.now()));
            
            logger.info("Order created successfully: {}", order.getOrderId());
            return order;
            
        } catch (Exception e) {
            eventMetrics.recordEventFailed("OrderCreatedEvent");
            logger.error("Error creating order", e);
            throw e;
        }
    }
}
```

---

## Summary of Part 4

**Key Takeaways:**
1. **Saga Pattern** manages distributed transactions with compensation
2. **Event Sourcing** stores all changes as events for audit and reconstruction
3. **Dead Letter Queues** handle messages that cannot be processed
4. **Retry mechanisms** with exponential backoff improve resilience
5. **Monitoring and observability** are crucial for distributed systems
6. **Idempotency** ensures safe event replay
7. **Performance optimization** through batching and async processing
8. **Best practices** ensure maintainable and scalable systems

**Complete Tutorial Summary:**
- **Part 1**: Fundamentals and architecture concepts
- **Part 2**: Infrastructure setup and configuration
- **Part 3**: Complete microservices implementation
- **Part 4**: Advanced patterns, monitoring, and best practices

This comprehensive tutorial covers building production-ready event-driven microservices in Java, from basics to advanced patterns. Apply these concepts to build scalable, resilient, and maintainable distributed systems!

