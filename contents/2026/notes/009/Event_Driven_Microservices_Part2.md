# Building Event-Driven Microservices in Java - Step by Step

## Part 2: Infrastructure Setup and Configuration

---

## Table of Contents

1. [Setting Up RabbitMQ](#1-setting-up-rabbitmq)
2. [Project Structure](#2-project-structure)
3. [Common Module Setup](#3-common-module-setup)
4. [Order Service Setup](#4-order-service-setup)
5. [Event Publisher Implementation](#5-event-publisher-implementation)
6. [Event Consumer Implementation](#6-event-consumer-implementation)
7. [Configuration Files](#7-configuration-files)

---

## 1. Setting Up RabbitMQ

### 1.1 Using Docker (Recommended)

```bash
# Run RabbitMQ with management UI
docker run -d \
  --name rabbitmq \
  -p 5672:5672 \
  -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin \
  -e RABBITMQ_DEFAULT_PASS=admin \
  rabbitmq:3-management

# Verify it's running
docker ps | grep rabbitmq
```

**Access Management UI:**
- URL: http://localhost:15672
- Username: admin
- Password: admin

### 1.2 RabbitMQ Concepts

**Exchange Types:**
- **Direct**: Routes messages to queues based on routing key
- **Topic**: Routes messages using pattern matching on routing key
- **Fanout**: Broadcasts messages to all bound queues
- **Headers**: Routes based on message headers

**Queue:**
- Stores messages until consumed
- Can be durable (survives broker restart)
- Can be exclusive (only one connection)

**Binding:**
- Links exchange to queue
- Can include routing key

### 1.3 RabbitMQ Setup Script

```bash
#!/bin/bash
# setup-rabbitmq.sh

# Create exchanges
rabbitmqadmin declare exchange name=order.exchange type=topic durable=true

# Create queues
rabbitmqadmin declare queue name=order.created.queue durable=true
rabbitmqadmin declare queue name=payment.processed.queue durable=true
rabbitmqadmin declare queue name=inventory.reserved.queue durable=true

# Create bindings
rabbitmqadmin declare binding source=order.exchange destination=order.created.queue routing_key=order.created
rabbitmqadmin declare binding source=order.exchange destination=payment.processed.queue routing_key=payment.processed
rabbitmqadmin declare binding source=order.exchange destination=inventory.reserved.queue routing_key=inventory.reserved
```

---

## 2. Project Structure

### 2.1 Multi-Module Maven Project

```
event-driven-microservices/
├── pom.xml (Parent POM)
├── common/
│   ├── pom.xml
│   └── src/main/java/com/example/common/
│       ├── event/
│       ├── config/
│       └── util/
├── order-service/
│   ├── pom.xml
│   └── src/main/java/com/example/order/
├── payment-service/
│   ├── pom.xml
│   └── src/main/java/com/example/payment/
├── inventory-service/
│   ├── pom.xml
│   └── src/main/java/com/example/inventory/
└── notification-service/
    ├── pom.xml
    └── src/main/java/com/example/notification/
```

### 2.2 Parent POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>event-driven-microservices</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>

    <modules>
        <module>common</module>
        <module>order-service</module>
        <module>payment-service</module>
        <module>inventory-service</module>
        <module>notification-service</module>
    </modules>

    <properties>
        <java.version>11</java.version>
        <spring-boot.version>2.7.0</spring-boot.version>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-maven-plugin</artifactId>
                    <version>${spring-boot.version}</version>
                </plugin>
            </plugins>
        </pluginManagement>
    </build>
</project>
```

---

## 3. Common Module Setup

### 3.1 Common POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>event-driven-microservices</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>common</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
    </dependencies>
</project>
```

### 3.2 Base Event Class

```java
package com.example.common.event;

import java.time.LocalDateTime;
import java.util.UUID;

public abstract class BaseEvent {
    private String eventId;
    private LocalDateTime timestamp;
    private String eventType;

    public BaseEvent() {
        this.eventId = UUID.randomUUID().toString();
        this.timestamp = LocalDateTime.now();
        this.eventType = this.getClass().getSimpleName();
    }

    public BaseEvent(String eventType) {
        this.eventId = UUID.randomUUID().toString();
        this.timestamp = LocalDateTime.now();
        this.eventType = eventType;
    }

    // Getters and setters
    public String getEventId() {
        return eventId;
    }

    public void setEventId(String eventId) {
        this.eventId = eventId;
    }

    public LocalDateTime getTimestamp() {
        return timestamp;
    }

    public void setTimestamp(LocalDateTime timestamp) {
        this.timestamp = timestamp;
    }

    public String getEventType() {
        return eventType;
    }

    public void setEventType(String eventType) {
        this.eventType = eventType;
    }
}
```

### 3.3 Order Events

```java
package com.example.common.event;

import java.math.BigDecimal;
import java.util.List;

public class OrderCreatedEvent extends BaseEvent {
    private String orderId;
    private String customerId;
    private List<OrderItem> items;
    private BigDecimal totalAmount;

    public OrderCreatedEvent() {
        super("OrderCreatedEvent");
    }

    public OrderCreatedEvent(String orderId, String customerId, 
                           List<OrderItem> items, BigDecimal totalAmount) {
        super("OrderCreatedEvent");
        this.orderId = orderId;
        this.customerId = customerId;
        this.items = items;
        this.totalAmount = totalAmount;
    }

    // Getters and setters
    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }

    public String getCustomerId() {
        return customerId;
    }

    public void setCustomerId(String customerId) {
        this.customerId = customerId;
    }

    public List<OrderItem> getItems() {
        return items;
    }

    public void setItems(List<OrderItem> items) {
        this.items = items;
    }

    public BigDecimal getTotalAmount() {
        return totalAmount;
    }

    public void setTotalAmount(BigDecimal totalAmount) {
        this.totalAmount = totalAmount;
    }
}

// OrderItem class
class OrderItem {
    private String productId;
    private int quantity;
    private BigDecimal price;

    public OrderItem() {}

    public OrderItem(String productId, int quantity, BigDecimal price) {
        this.productId = productId;
        this.quantity = quantity;
        this.price = price;
    }

    // Getters and setters
    public String getProductId() {
        return productId;
    }

    public void setProductId(String productId) {
        this.productId = productId;
    }

    public int getQuantity() {
        return quantity;
    }

    public void setQuantity(int quantity) {
        this.quantity = quantity;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }
}
```

### 3.4 Payment Events

```java
package com.example.common.event;

import java.math.BigDecimal;

public class PaymentProcessedEvent extends BaseEvent {
    private String paymentId;
    private String orderId;
    private String customerId;
    private BigDecimal amount;
    private String paymentMethod;
    private PaymentStatus status;

    public PaymentProcessedEvent() {
        super("PaymentProcessedEvent");
    }

    public PaymentProcessedEvent(String paymentId, String orderId, 
                                String customerId, BigDecimal amount, 
                                String paymentMethod, PaymentStatus status) {
        super("PaymentProcessedEvent");
        this.paymentId = paymentId;
        this.orderId = orderId;
        this.customerId = customerId;
        this.amount = amount;
        this.paymentMethod = paymentMethod;
        this.status = status;
    }

    // Getters and setters
    public String getPaymentId() {
        return paymentId;
    }

    public void setPaymentId(String paymentId) {
        this.paymentId = paymentId;
    }

    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }

    public String getCustomerId() {
        return customerId;
    }

    public void setCustomerId(String customerId) {
        this.customerId = customerId;
    }

    public BigDecimal getAmount() {
        return amount;
    }

    public void setAmount(BigDecimal amount) {
        this.amount = amount;
    }

    public String getPaymentMethod() {
        return paymentMethod;
    }

    public void setPaymentMethod(String paymentMethod) {
        this.paymentMethod = paymentMethod;
    }

    public PaymentStatus getStatus() {
        return status;
    }

    public void setStatus(PaymentStatus status) {
        this.status = status;
    }
}

enum PaymentStatus {
    SUCCESS, FAILED, PENDING, REFUNDED
}
```

### 3.5 Inventory Events

```java
package com.example.common.event;

import java.util.List;

public class InventoryReservedEvent extends BaseEvent {
    private String orderId;
    private List<InventoryItem> reservedItems;
    private boolean success;

    public InventoryReservedEvent() {
        super("InventoryReservedEvent");
    }

    public InventoryReservedEvent(String orderId, List<InventoryItem> reservedItems, boolean success) {
        super("InventoryReservedEvent");
        this.orderId = orderId;
        this.reservedItems = reservedItems;
        this.success = success;
    }

    // Getters and setters
    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }

    public List<InventoryItem> getReservedItems() {
        return reservedItems;
    }

    public void setReservedItems(List<InventoryItem> reservedItems) {
        this.reservedItems = reservedItems;
    }

    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }
}

class InventoryItem {
    private String productId;
    private int quantity;

    public InventoryItem() {}

    public InventoryItem(String productId, int quantity) {
        this.productId = productId;
        this.quantity = quantity;
    }

    // Getters and setters
    public String getProductId() {
        return productId;
    }

    public void setProductId(String productId) {
        this.productId = productId;
    }

    public int getQuantity() {
        return quantity;
    }

    public void setQuantity(int quantity) {
        this.quantity = quantity;
    }
}
```

### 3.6 RabbitMQ Configuration

```java
package com.example.common.config;

import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.support.converter.Jackson2JsonMessageConverter;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class RabbitMQConfig {

    // Exchange
    @Bean
    public TopicExchange orderExchange() {
        return new TopicExchange("order.exchange", true, false);
    }

    // Queues
    @Bean
    public Queue orderCreatedQueue() {
        return QueueBuilder.durable("order.created.queue").build();
    }

    @Bean
    public Queue paymentProcessedQueue() {
        return QueueBuilder.durable("payment.processed.queue").build();
    }

    @Bean
    public Queue inventoryReservedQueue() {
        return QueueBuilder.durable("inventory.reserved.queue").build();
    }

    // Bindings
    @Bean
    public Binding orderCreatedBinding() {
        return BindingBuilder
            .bind(orderCreatedQueue())
            .to(orderExchange())
            .with("order.created");
    }

    @Bean
    public Binding paymentProcessedBinding() {
        return BindingBuilder
            .bind(paymentProcessedQueue())
            .to(orderExchange())
            .with("payment.processed");
    }

    @Bean
    public Binding inventoryReservedBinding() {
        return BindingBuilder
            .bind(inventoryReservedQueue())
            .to(orderExchange())
            .with("inventory.reserved");
    }

    // JSON Message Converter
    @Bean
    public Jackson2JsonMessageConverter messageConverter() {
        return new Jackson2JsonMessageConverter();
    }

    // RabbitTemplate
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        RabbitTemplate template = new RabbitTemplate(connectionFactory);
        template.setMessageConverter(messageConverter());
        return template;
    }
}
```

---

## 4. Order Service Setup

### 4.1 Order Service POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
         http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <parent>
        <groupId>com.example</groupId>
        <artifactId>event-driven-microservices</artifactId>
        <version>1.0.0</version>
    </parent>

    <artifactId>order-service</artifactId>
    <packaging>jar</packaging>

    <dependencies>
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>common</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

### 4.2 Order Model

```java
package com.example.order.model;

import javax.persistence.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;
import java.util.List;

@Entity
@Table(name = "orders")
public class Order {
    @Id
    private String orderId;
    
    private String customerId;
    
    @Enumerated(EnumType.STRING)
    private OrderStatus status;
    
    private BigDecimal totalAmount;
    
    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinColumn(name = "order_id")
    private List<OrderItem> items;
    
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;

    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }

    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }

    // Getters and setters
    public String getOrderId() {
        return orderId;
    }

    public void setOrderId(String orderId) {
        this.orderId = orderId;
    }

    public String getCustomerId() {
        return customerId;
    }

    public void setCustomerId(String customerId) {
        this.customerId = customerId;
    }

    public OrderStatus getStatus() {
        return status;
    }

    public void setStatus(OrderStatus status) {
        this.status = status;
    }

    public BigDecimal getTotalAmount() {
        return totalAmount;
    }

    public void setTotalAmount(BigDecimal totalAmount) {
        this.totalAmount = totalAmount;
    }

    public List<OrderItem> getItems() {
        return items;
    }

    public void setItems(List<OrderItem> items) {
        this.items = items;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(LocalDateTime createdAt) {
        this.createdAt = createdAt;
    }

    public LocalDateTime getUpdatedAt() {
        return updatedAt;
    }

    public void setUpdatedAt(LocalDateTime updatedAt) {
        this.updatedAt = updatedAt;
    }
}

@Entity
@Table(name = "order_items")
class OrderItem {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String productId;
    private int quantity;
    private BigDecimal price;

    // Getters and setters
    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getProductId() {
        return productId;
    }

    public void setProductId(String productId) {
        this.productId = productId;
    }

    public int getQuantity() {
        return quantity;
    }

    public void setQuantity(int quantity) {
        this.quantity = quantity;
    }

    public BigDecimal getPrice() {
        return price;
    }

    public void setPrice(BigDecimal price) {
        this.price = price;
    }
}

enum OrderStatus {
    CREATED, PROCESSING, PAID, SHIPPED, DELIVERED, CANCELLED
}
```

---

## 5. Event Publisher Implementation

### 5.1 Event Publisher Interface

```java
package com.example.common.event;

public interface EventPublisher {
    void publish(String routingKey, BaseEvent event);
}
```

### 5.2 RabbitMQ Event Publisher

```java
package com.example.common.event;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class RabbitMQEventPublisher implements EventPublisher {
    
    private static final Logger logger = LoggerFactory.getLogger(RabbitMQEventPublisher.class);
    
    private final RabbitTemplate rabbitTemplate;
    private static final String EXCHANGE_NAME = "order.exchange";
    
    @Autowired
    public RabbitMQEventPublisher(RabbitTemplate rabbitTemplate) {
        this.rabbitTemplate = rabbitTemplate;
    }
    
    @Override
    public void publish(String routingKey, BaseEvent event) {
        try {
            logger.info("Publishing event: {} with routing key: {}", event.getEventType(), routingKey);
            rabbitTemplate.convertAndSend(EXCHANGE_NAME, routingKey, event);
            logger.info("Event published successfully: {}", event.getEventId());
        } catch (Exception e) {
            logger.error("Error publishing event: {}", event.getEventId(), e);
            throw new EventPublishException("Failed to publish event", e);
        }
    }
}

class EventPublishException extends RuntimeException {
    public EventPublishException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

---

## 6. Event Consumer Implementation

### 6.1 Event Listener Base

```java
package com.example.common.event;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Component;

@Component
public abstract class EventListener<T extends BaseEvent> {
    
    private static final Logger logger = LoggerFactory.getLogger(EventListener.class);
    
    @RabbitListener(queues = "#{@getQueueName()}")
    public void handleEvent(T event) {
        try {
            logger.info("Received event: {} with ID: {}", event.getEventType(), event.getEventId());
            processEvent(event);
            logger.info("Event processed successfully: {}", event.getEventId());
        } catch (Exception e) {
            logger.error("Error processing event: {}", event.getEventId(), e);
            handleError(event, e);
        }
    }
    
    protected abstract void processEvent(T event);
    
    protected void handleError(T event, Exception e) {
        // Default error handling - can be overridden
        logger.error("Event processing failed, event will be retried or sent to DLQ", e);
    }
}
```

### 6.2 Example Event Consumer

```java
package com.example.payment.listener;

import com.example.common.event.BaseEvent;
import com.example.common.event.OrderCreatedEvent;
import com.example.common.event.EventListener;
import org.springframework.stereotype.Component;

@Component
public class OrderCreatedEventListener extends EventListener<OrderCreatedEvent> {
    
    @Override
    protected void processEvent(OrderCreatedEvent event) {
        // Process payment
        System.out.println("Processing payment for order: " + event.getOrderId());
        // Implementation here
    }
}
```

---

## 7. Configuration Files

### 7.1 Application Properties (Order Service)

```yaml
# application.yml
server:
  port: 8081

spring:
  application:
    name: order-service
  
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin
    virtual-host: /
    
  datasource:
    url: jdbc:h2:mem:orderdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  
  h2:
    console:
      enabled: true
      path: /h2-console
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true
    properties:
      hibernate:
        format_sql: true

logging:
  level:
    com.example: DEBUG
    org.springframework.amqp: DEBUG
```

### 7.2 Application Properties (Payment Service)

```yaml
# application.yml
server:
  port: 8082

spring:
  application:
    name: payment-service
  
  rabbitmq:
    host: localhost
    port: 5672
    username: admin
    password: admin
    virtual-host: /
    
  datasource:
    url: jdbc:h2:mem:paymentdb
    driver-class-name: org.h2.Driver
    username: sa
    password: 
  
  h2:
    console:
      enabled: true
      path: /h2-console
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

logging:
  level:
    com.example: DEBUG
```

---

## Summary of Part 2

**Key Takeaways:**
1. **RabbitMQ Setup** using Docker for easy development
2. **Multi-module Maven project** structure for microservices
3. **Common module** for shared events and configurations
4. **Event classes** extend BaseEvent for consistency
5. **RabbitMQ configuration** with exchanges, queues, and bindings
6. **Event Publisher** for publishing events
7. **Event Consumer** base class for handling events
8. **Configuration files** for each service

**Next Steps:**
- Part 3 will implement complete microservices with full business logic
- Part 4 will cover advanced patterns (Saga, Event Sourcing, monitoring)

**Commands to Run:**
```bash
# Start RabbitMQ
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 \
  -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin \
  rabbitmq:3-management

# Build project
mvn clean install

# Run services
cd order-service && mvn spring-boot:run
cd payment-service && mvn spring-boot:run
```

