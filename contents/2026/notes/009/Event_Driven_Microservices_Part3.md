# Building Event-Driven Microservices in Java - Step by Step

## Part 3: Complete Microservices Implementation

---

## Table of Contents

1. [Order Service Implementation](#1-order-service-implementation)
2. [Payment Service Implementation](#2-payment-service-implementation)
3. [Inventory Service Implementation](#3-inventory-service-implementation)
4. [Notification Service Implementation](#4-notification-service-implementation)
5. [Testing the System](#5-testing-the-system)
6. [Event Flow Example](#6-event-flow-example)

---

## 1. Order Service Implementation

### 1.1 Order Repository

```java
package com.example.order.repository;

import com.example.order.model.Order;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface OrderRepository extends JpaRepository<Order, String> {
    List<Order> findByCustomerId(String customerId);
    Optional<Order> findByOrderId(String orderId);
}
```

### 1.2 Order Service

```java
package com.example.order.service;

import com.example.common.event.EventPublisher;
import com.example.common.event.OrderCreatedEvent;
import com.example.order.model.Order;
import com.example.order.model.OrderItem;
import com.example.order.model.OrderStatus;
import com.example.order.repository.OrderRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.List;
import java.util.UUID;
import java.util.stream.Collectors;

@Service
public class OrderService {
    
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    private final OrderRepository orderRepository;
    private final EventPublisher eventPublisher;
    
    @Autowired
    public OrderService(OrderRepository orderRepository, EventPublisher eventPublisher) {
        this.orderRepository = orderRepository;
        this.eventPublisher = eventPublisher;
    }
    
    @Transactional
    public Order createOrder(CreateOrderRequest request) {
        logger.info("Creating order for customer: {}", request.getCustomerId());
        
        // Create order
        Order order = new Order();
        order.setOrderId(UUID.randomUUID().toString());
        order.setCustomerId(request.getCustomerId());
        order.setStatus(OrderStatus.CREATED);
        
        // Convert request items to order items
        List<OrderItem> orderItems = request.getItems().stream()
            .map(item -> {
                OrderItem orderItem = new OrderItem();
                orderItem.setProductId(item.getProductId());
                orderItem.setQuantity(item.getQuantity());
                orderItem.setPrice(item.getPrice());
                return orderItem;
            })
            .collect(Collectors.toList());
        
        order.setItems(orderItems);
        
        // Calculate total
        BigDecimal total = orderItems.stream()
            .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
            .reduce(BigDecimal.ZERO, BigDecimal::add);
        
        order.setTotalAmount(total);
        
        // Save order
        order = orderRepository.save(order);
        logger.info("Order created: {}", order.getOrderId());
        
        // Publish event
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getOrderId(),
            order.getCustomerId(),
            convertToEventItems(orderItems),
            order.getTotalAmount()
        );
        
        eventPublisher.publish("order.created", event);
        logger.info("OrderCreatedEvent published for order: {}", order.getOrderId());
        
        return order;
    }
    
    public Order getOrder(String orderId) {
        return orderRepository.findByOrderId(orderId)
            .orElseThrow(() -> new OrderNotFoundException("Order not found: " + orderId));
    }
    
    public List<Order> getOrdersByCustomer(String customerId) {
        return orderRepository.findByCustomerId(customerId);
    }
    
    @Transactional
    public void updateOrderStatus(String orderId, OrderStatus status) {
        Order order = getOrder(orderId);
        order.setStatus(status);
        orderRepository.save(order);
        logger.info("Order status updated: {} -> {}", orderId, status);
    }
    
    private List<com.example.common.event.OrderItem> convertToEventItems(List<OrderItem> items) {
        return items.stream()
            .map(item -> new com.example.common.event.OrderItem(
                item.getProductId(),
                item.getQuantity(),
                item.getPrice()
            ))
            .collect(Collectors.toList());
    }
}

// Request DTO
class CreateOrderRequest {
    private String customerId;
    private List<OrderItemRequest> items;
    
    // Getters and setters
    public String getCustomerId() {
        return customerId;
    }
    
    public void setCustomerId(String customerId) {
        this.customerId = customerId;
    }
    
    public List<OrderItemRequest> getItems() {
        return items;
    }
    
    public void setItems(List<OrderItemRequest> items) {
        this.items = items;
    }
}

class OrderItemRequest {
    private String productId;
    private int quantity;
    private BigDecimal price;
    
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

class OrderNotFoundException extends RuntimeException {
    public OrderNotFoundException(String message) {
        super(message);
    }
}
```

### 1.3 Order Controller

```java
package com.example.order.controller;

import com.example.order.model.Order;
import com.example.order.service.OrderService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/orders")
public class OrderController {
    
    private final OrderService orderService;
    
    @Autowired
    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }
    
    @PostMapping
    public ResponseEntity<Order> createOrder(@RequestBody CreateOrderRequest request) {
        Order order = orderService.createOrder(request);
        return ResponseEntity.status(HttpStatus.CREATED).body(order);
    }
    
    @GetMapping("/{orderId}")
    public ResponseEntity<Order> getOrder(@PathVariable String orderId) {
        Order order = orderService.getOrder(orderId);
        return ResponseEntity.ok(order);
    }
    
    @GetMapping("/customer/{customerId}")
    public ResponseEntity<List<Order>> getOrdersByCustomer(@PathVariable String customerId) {
        List<Order> orders = orderService.getOrdersByCustomer(customerId);
        return ResponseEntity.ok(orders);
    }
}

// Request DTO (same as in service)
class CreateOrderRequest {
    private String customerId;
    private List<OrderItemRequest> items;
    
    // Getters and setters
    public String getCustomerId() {
        return customerId;
    }
    
    public void setCustomerId(String customerId) {
        this.customerId = customerId;
    }
    
    public List<OrderItemRequest> getItems() {
        return items;
    }
    
    public void setItems(List<OrderItemRequest> items) {
        this.items = items;
    }
}

class OrderItemRequest {
    private String productId;
    private int quantity;
    private java.math.BigDecimal price;
    
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
    
    public java.math.BigDecimal getPrice() {
        return price;
    }
    
    public void setPrice(java.math.BigDecimal price) {
        this.price = price;
    }
}
```

### 1.4 Order Application

```java
package com.example.order;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class OrderApplication {
    public static void main(String[] args) {
        SpringApplication.run(OrderApplication.class, args);
    }
}
```

---

## 2. Payment Service Implementation

### 2.1 Payment Model

```java
package com.example.payment.model;

import javax.persistence.*;
import java.math.BigDecimal;
import java.time.LocalDateTime;

@Entity
@Table(name = "payments")
public class Payment {
    @Id
    private String paymentId;
    
    private String orderId;
    private String customerId;
    private BigDecimal amount;
    private String paymentMethod;
    
    @Enumerated(EnumType.STRING)
    private PaymentStatus status;
    
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private String failureReason;

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

    public String getFailureReason() {
        return failureReason;
    }

    public void setFailureReason(String failureReason) {
        this.failureReason = failureReason;
    }
}

enum PaymentStatus {
    PENDING, SUCCESS, FAILED, REFUNDED
}
```

### 2.2 Payment Repository

```java
package com.example.payment.repository;

import com.example.payment.model.Payment;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Repository
public interface PaymentRepository extends JpaRepository<Payment, String> {
    Optional<Payment> findByOrderId(String orderId);
    List<Payment> findByCustomerId(String customerId);
}
```

### 2.3 Payment Service

```java
package com.example.payment.service;

import com.example.common.event.EventPublisher;
import com.example.common.event.OrderCreatedEvent;
import com.example.common.event.PaymentProcessedEvent;
import com.example.common.event.PaymentStatus;
import com.example.payment.model.Payment;
import com.example.payment.model.PaymentStatus as ModelPaymentStatus;
import com.example.payment.repository.PaymentRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.math.BigDecimal;
import java.util.UUID;

@Service
public class PaymentService {
    
    private static final Logger logger = LoggerFactory.getLogger(PaymentService.class);
    
    private final PaymentRepository paymentRepository;
    private final EventPublisher eventPublisher;
    
    @Autowired
    public PaymentService(PaymentRepository paymentRepository, EventPublisher eventPublisher) {
        this.paymentRepository = paymentRepository;
        this.eventPublisher = eventPublisher;
    }
    
    @Transactional
    public void processPayment(OrderCreatedEvent event) {
        logger.info("Processing payment for order: {}", event.getOrderId());
        
        // Create payment record
        Payment payment = new Payment();
        payment.setPaymentId(UUID.randomUUID().toString());
        payment.setOrderId(event.getOrderId());
        payment.setCustomerId(event.getCustomerId());
        payment.setAmount(event.getTotalAmount());
        payment.setPaymentMethod("CREDIT_CARD"); // Default
        payment.setStatus(ModelPaymentStatus.PENDING);
        
        payment = paymentRepository.save(payment);
        
        // Simulate payment processing
        boolean paymentSuccess = processPaymentInternal(payment);
        
        if (paymentSuccess) {
            payment.setStatus(ModelPaymentStatus.SUCCESS);
            logger.info("Payment successful for order: {}", event.getOrderId());
        } else {
            payment.setStatus(ModelPaymentStatus.FAILED);
            payment.setFailureReason("Insufficient funds");
            logger.warn("Payment failed for order: {}", event.getOrderId());
        }
        
        payment = paymentRepository.save(payment);
        
        // Publish payment processed event
        PaymentProcessedEvent paymentEvent = new PaymentProcessedEvent(
            payment.getPaymentId(),
            payment.getOrderId(),
            payment.getCustomerId(),
            payment.getAmount(),
            payment.getPaymentMethod(),
            paymentSuccess ? PaymentStatus.SUCCESS : PaymentStatus.FAILED
        );
        
        eventPublisher.publish("payment.processed", paymentEvent);
        logger.info("PaymentProcessedEvent published for order: {}", event.getOrderId());
    }
    
    private boolean processPaymentInternal(Payment payment) {
        // Simulate payment gateway call
        // In real implementation, this would call a payment gateway API
        try {
            Thread.sleep(100); // Simulate network delay
            // Simulate 90% success rate
            return Math.random() > 0.1;
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
            return false;
        }
    }
    
    public Payment getPayment(String paymentId) {
        return paymentRepository.findById(paymentId)
            .orElseThrow(() -> new PaymentNotFoundException("Payment not found: " + paymentId));
    }
    
    public Payment getPaymentByOrderId(String orderId) {
        return paymentRepository.findByOrderId(orderId)
            .orElseThrow(() -> new PaymentNotFoundException("Payment not found for order: " + orderId));
    }
}

class PaymentNotFoundException extends RuntimeException {
    public PaymentNotFoundException(String message) {
        super(message);
    }
}
```

### 2.4 Payment Event Listener

```java
package com.example.payment.listener;

import com.example.common.event.OrderCreatedEvent;
import com.example.payment.service.PaymentService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class PaymentEventListener {
    
    private static final Logger logger = LoggerFactory.getLogger(PaymentEventListener.class);
    
    private final PaymentService paymentService;
    
    @Autowired
    public PaymentEventListener(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
    
    @RabbitListener(queues = "order.created.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        logger.info("Received OrderCreatedEvent: {}", event.getOrderId());
        try {
            paymentService.processPayment(event);
        } catch (Exception e) {
            logger.error("Error processing payment for order: {}", event.getOrderId(), e);
            throw e; // Will trigger retry or DLQ
        }
    }
}
```

### 2.5 Payment Controller

```java
package com.example.payment.controller;

import com.example.payment.model.Payment;
import com.example.payment.service.PaymentService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/payments")
public class PaymentController {
    
    private final PaymentService paymentService;
    
    @Autowired
    public PaymentController(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
    
    @GetMapping("/{paymentId}")
    public ResponseEntity<Payment> getPayment(@PathVariable String paymentId) {
        Payment payment = paymentService.getPayment(paymentId);
        return ResponseEntity.ok(payment);
    }
    
    @GetMapping("/order/{orderId}")
    public ResponseEntity<Payment> getPaymentByOrder(@PathVariable String orderId) {
        Payment payment = paymentService.getPaymentByOrderId(orderId);
        return ResponseEntity.ok(payment);
    }
}
```

---

## 3. Inventory Service Implementation

### 3.1 Inventory Model

```java
package com.example.inventory.model;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "inventory")
public class Inventory {
    @Id
    private String productId;
    
    private int availableStock;
    private int reservedStock;
    private int totalStock;
    
    private LocalDateTime lastUpdated;

    @PrePersist
    @PreUpdate
    protected void onUpdate() {
        lastUpdated = LocalDateTime.now();
    }

    // Getters and setters
    public String getProductId() {
        return productId;
    }

    public void setProductId(String productId) {
        this.productId = productId;
    }

    public int getAvailableStock() {
        return availableStock;
    }

    public void setAvailableStock(int availableStock) {
        this.availableStock = availableStock;
    }

    public int getReservedStock() {
        return reservedStock;
    }

    public void setReservedStock(int reservedStock) {
        this.reservedStock = reservedStock;
    }

    public int getTotalStock() {
        return totalStock;
    }

    public void setTotalStock(int totalStock) {
        this.totalStock = totalStock;
    }

    public LocalDateTime getLastUpdated() {
        return lastUpdated;
    }

    public void setLastUpdated(LocalDateTime lastUpdated) {
        this.lastUpdated = lastUpdated;
    }
}
```

### 3.2 Inventory Repository

```java
package com.example.inventory.repository;

import com.example.inventory.model.Inventory;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.Optional;

@Repository
public interface InventoryRepository extends JpaRepository<Inventory, String> {
    Optional<Inventory> findByProductId(String productId);
}
```

### 3.3 Inventory Service

```java
package com.example.inventory.service;

import com.example.common.event.EventPublisher;
import com.example.common.event.InventoryReservedEvent;
import com.example.common.event.OrderCreatedEvent;
import com.example.common.event.OrderItem;
import com.example.inventory.model.Inventory;
import com.example.inventory.repository.InventoryRepository;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

@Service
public class InventoryService {
    
    private static final Logger logger = LoggerFactory.getLogger(InventoryService.class);
    
    private final InventoryRepository inventoryRepository;
    private final EventPublisher eventPublisher;
    
    @Autowired
    public InventoryService(InventoryRepository inventoryRepository, EventPublisher eventPublisher) {
        this.inventoryRepository = inventoryRepository;
        this.eventPublisher = eventPublisher;
    }
    
    @Transactional
    public void reserveInventory(OrderCreatedEvent event) {
        logger.info("Reserving inventory for order: {}", event.getOrderId());
        
        List<com.example.common.event.InventoryItem> reservedItems = new ArrayList<>();
        boolean allReserved = true;
        
        for (OrderItem orderItem : event.getItems()) {
            String productId = orderItem.getProductId();
            int quantity = orderItem.getQuantity();
            
            Inventory inventory = inventoryRepository.findByProductId(productId)
                .orElseGet(() -> {
                    Inventory newInventory = new Inventory();
                    newInventory.setProductId(productId);
                    newInventory.setTotalStock(0);
                    newInventory.setAvailableStock(0);
                    newInventory.setReservedStock(0);
                    return newInventory;
                });
            
            if (inventory.getAvailableStock() >= quantity) {
                // Reserve stock
                inventory.setAvailableStock(inventory.getAvailableStock() - quantity);
                inventory.setReservedStock(inventory.getReservedStock() + quantity);
                inventoryRepository.save(inventory);
                
                reservedItems.add(new com.example.common.event.InventoryItem(productId, quantity));
                logger.info("Reserved {} units of product {}", quantity, productId);
            } else {
                logger.warn("Insufficient stock for product: {}. Available: {}, Required: {}", 
                    productId, inventory.getAvailableStock(), quantity);
                allReserved = false;
                break;
            }
        }
        
        // Publish inventory reserved event
        InventoryReservedEvent inventoryEvent = new InventoryReservedEvent(
            event.getOrderId(),
            reservedItems,
            allReserved
        );
        
        eventPublisher.publish("inventory.reserved", inventoryEvent);
        logger.info("InventoryReservedEvent published for order: {}", event.getOrderId());
    }
    
    public Inventory getInventory(String productId) {
        return inventoryRepository.findByProductId(productId)
            .orElseThrow(() -> new InventoryNotFoundException("Inventory not found: " + productId));
    }
    
    @Transactional
    public void addStock(String productId, int quantity) {
        Inventory inventory = inventoryRepository.findByProductId(productId)
            .orElseGet(() -> {
                Inventory newInventory = new Inventory();
                newInventory.setProductId(productId);
                newInventory.setTotalStock(0);
                newInventory.setAvailableStock(0);
                newInventory.setReservedStock(0);
                return newInventory;
            });
        
        inventory.setTotalStock(inventory.getTotalStock() + quantity);
        inventory.setAvailableStock(inventory.getAvailableStock() + quantity);
        inventoryRepository.save(inventory);
        
        logger.info("Added {} units to product {}. Total stock: {}", 
            quantity, productId, inventory.getTotalStock());
    }
}

class InventoryNotFoundException extends RuntimeException {
    public InventoryNotFoundException(String message) {
        super(message);
    }
}
```

### 3.4 Inventory Event Listener

```java
package com.example.inventory.listener;

import com.example.common.event.OrderCreatedEvent;
import com.example.inventory.service.InventoryService;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class InventoryEventListener {
    
    private static final Logger logger = LoggerFactory.getLogger(InventoryEventListener.class);
    
    private final InventoryService inventoryService;
    
    @Autowired
    public InventoryEventListener(InventoryService inventoryService) {
        this.inventoryService = inventoryService;
    }
    
    @RabbitListener(queues = "order.created.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        logger.info("Received OrderCreatedEvent for inventory: {}", event.getOrderId());
        try {
            inventoryService.reserveInventory(event);
        } catch (Exception e) {
            logger.error("Error reserving inventory for order: {}", event.getOrderId(), e);
            throw e;
        }
    }
}
```

### 3.5 Inventory Controller

```java
package com.example.inventory.controller;

import com.example.inventory.model.Inventory;
import com.example.inventory.service.InventoryService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/inventory")
public class InventoryController {
    
    private final InventoryService inventoryService;
    
    @Autowired
    public InventoryController(InventoryService inventoryService) {
        this.inventoryService = inventoryService;
    }
    
    @GetMapping("/{productId}")
    public ResponseEntity<Inventory> getInventory(@PathVariable String productId) {
        Inventory inventory = inventoryService.getInventory(productId);
        return ResponseEntity.ok(inventory);
    }
    
    @PostMapping("/{productId}/stock")
    public ResponseEntity<Void> addStock(
            @PathVariable String productId,
            @RequestParam int quantity) {
        inventoryService.addStock(productId, quantity);
        return ResponseEntity.ok().build();
    }
}
```

---

## 4. Notification Service Implementation

### 4.1 Notification Service

```java
package com.example.notification.service;

import com.example.common.event.*;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.amqp.rabbit.annotation.RabbitListener;
import org.springframework.stereotype.Service;

@Service
public class NotificationService {
    
    private static final Logger logger = LoggerFactory.getLogger(NotificationService.class);
    
    @RabbitListener(queues = "order.created.queue")
    public void handleOrderCreated(OrderCreatedEvent event) {
        logger.info("Sending order confirmation email to customer: {}", event.getCustomerId());
        sendEmail(event.getCustomerId(), 
            "Order Confirmation", 
            "Your order " + event.getOrderId() + " has been created.");
    }
    
    @RabbitListener(queues = "payment.processed.queue")
    public void handlePaymentProcessed(PaymentProcessedEvent event) {
        if (event.getStatus() == PaymentStatus.SUCCESS) {
            logger.info("Sending payment confirmation email to customer: {}", event.getCustomerId());
            sendEmail(event.getCustomerId(),
                "Payment Confirmed",
                "Payment for order " + event.getOrderId() + " has been processed successfully.");
        } else {
            logger.info("Sending payment failure email to customer: {}", event.getCustomerId());
            sendEmail(event.getCustomerId(),
                "Payment Failed",
                "Payment for order " + event.getOrderId() + " has failed. Please try again.");
        }
    }
    
    @RabbitListener(queues = "inventory.reserved.queue")
    public void handleInventoryReserved(InventoryReservedEvent event) {
        if (event.isSuccess()) {
            logger.info("Inventory reserved for order: {}", event.getOrderId());
        } else {
            logger.warn("Inventory reservation failed for order: {}", event.getOrderId());
            sendEmail("admin@example.com",
                "Inventory Reservation Failed",
                "Failed to reserve inventory for order " + event.getOrderId());
        }
    }
    
    private void sendEmail(String to, String subject, String body) {
        // In real implementation, this would call an email service
        logger.info("Email sent - To: {}, Subject: {}, Body: {}", to, subject, body);
    }
}
```

### 4.2 Notification Application

```java
package com.example.notification;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class NotificationApplication {
    public static void main(String[] args) {
        SpringApplication.run(NotificationApplication.class, args);
    }
}
```

---

## 5. Testing the System

### 5.1 Create Order Request

```bash
# Create an order
curl -X POST http://localhost:8081/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "customerId": "customer-123",
    "items": [
      {
        "productId": "product-1",
        "quantity": 2,
        "price": 29.99
      },
      {
        "productId": "product-2",
        "quantity": 1,
        "price": 49.99
      }
    ]
  }'
```

### 5.2 Check Order Status

```bash
# Get order by ID
curl http://localhost:8081/api/orders/{orderId}

# Get orders by customer
curl http://localhost:8081/api/orders/customer/customer-123
```

### 5.3 Check Payment Status

```bash
# Get payment by order ID
curl http://localhost:8082/api/payments/order/{orderId}
```

### 5.4 Check Inventory

```bash
# Get inventory for a product
curl http://localhost:8083/api/inventory/{productId}

# Add stock
curl -X POST http://localhost:8083/api/inventory/{productId}/stock?quantity=100
```

### 5.5 Monitor RabbitMQ

1. Open http://localhost:15672
2. Login with admin/admin
3. Check queues for messages
4. Monitor message flow

---

## 6. Event Flow Example

### 6.1 Complete Flow

```
1. Client sends POST /api/orders
   ↓
2. Order Service creates order
   ↓
3. Order Service publishes OrderCreatedEvent
   ↓
4. RabbitMQ routes event to queues:
   ├─► order.created.queue
   │   ├─► Payment Service (processes payment)
   │   ├─► Inventory Service (reserves inventory)
   │   └─► Notification Service (sends confirmation)
   ↓
5. Payment Service publishes PaymentProcessedEvent
   ↓
6. RabbitMQ routes to payment.processed.queue
   └─► Notification Service (sends payment confirmation)
   ↓
7. Inventory Service publishes InventoryReservedEvent
   ↓
8. RabbitMQ routes to inventory.reserved.queue
   └─► Notification Service (sends inventory status)
```

### 6.2 Sequence Diagram

```
Client    Order Service    RabbitMQ    Payment Service    Inventory Service    Notification Service
  │            │              │              │                    │                      │
  │──POST─────►│              │              │                    │                      │
  │            │              │              │                    │                      │
  │            │──Create Order│              │                    │                      │
  │            │              │              │                    │                      │
  │            │──Publish────►│              │                    │                      │
  │            │  OrderCreated│              │                    │                      │
  │            │              │              │                    │                      │
  │            │              │──Event─────►│                    │                      │
  │            │              │              │                    │                      │
  │            │              │──Event──────────────────────────►│                      │
  │            │              │              │                    │                      │
  │            │              │──Event───────────────────────────────────────────────►│
  │            │              │              │                    │                      │
  │            │              │              │──Process Payment   │                      │
  │            │              │              │                    │                      │
  │            │              │◄──Publish────│                    │                      │
  │            │              │PaymentProcess│                    │                      │
  │            │              │              │                    │                      │
  │            │              │──Event───────────────────────────────────────────────►│
  │            │              │              │                    │                      │
  │            │              │              │                    │──Reserve Inventory   │
  │            │              │              │                    │                      │
  │            │              │◄──Publish────│                    │                      │
  │            │              │InventoryReserved                  │                      │
  │            │              │              │                    │                      │
  │            │              │──Event───────────────────────────────────────────────►│
  │            │              │              │                    │                      │
  │◄──Response─│              │              │                    │                      │
```

---

## Summary of Part 3

**Key Takeaways:**
1. **Order Service** creates orders and publishes OrderCreatedEvent
2. **Payment Service** listens for orders and processes payments
3. **Inventory Service** reserves inventory when orders are created
4. **Notification Service** sends notifications for all events
5. **Event-driven flow** enables loose coupling between services
6. **Each service** is independent and can scale separately

**Next Steps:**
- Part 4 will cover advanced topics: Saga pattern, Event Sourcing, monitoring, error handling, and best practices

**To Run:**
```bash
# Terminal 1: Order Service
cd order-service && mvn spring-boot:run

# Terminal 2: Payment Service
cd payment-service && mvn spring-boot:run

# Terminal 3: Inventory Service
cd inventory-service && mvn spring-boot:run

# Terminal 4: Notification Service
cd notification-service && mvn spring-boot:run
```

