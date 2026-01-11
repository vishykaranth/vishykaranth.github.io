# Java Design Patterns: Complete Guide with Examples

## Part 3: Structural Patterns - Adapter, Decorator, Facade

---

## Table of Contents

1. [Adapter Pattern](#1-adapter-pattern)
2. [Decorator Pattern](#2-decorator-pattern)
3. [Facade Pattern](#3-facade-pattern)

---

## 1. Adapter Pattern

### 1.1 Intent

Convert the interface of a class into another interface clients expect. Allows classes with incompatible interfaces to work together.

### 1.2 When to Use

- Need to use existing class with incompatible interface
- Want to integrate third-party libraries
- Need to work with legacy code
- Want to create reusable class that cooperates with unrelated classes

### 1.3 Scenarios

- **Payment Integration**: Adapt different payment gateways to common interface
- **Media Players**: Adapt different media formats to common player interface
- **Legacy Systems**: Integrate old systems with new code
- **API Integration**: Adapt external APIs to internal interfaces

### 1.4 Implementation: Class Adapter

```java
// Target interface (what client expects)
interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Adaptee (existing incompatible interface)
class AdvancedMediaPlayer {
    public void playVlc(String fileName) {
        System.out.println("Playing VLC file: " + fileName);
    }
    
    public void playMp4(String fileName) {
        System.out.println("Playing MP4 file: " + fileName);
    }
}

// Adapter (adapts Adaptee to Target)
class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;
    
    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer = new AdvancedMediaPlayer();
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer = new AdvancedMediaPlayer();
        }
    }
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer.playMp4(fileName);
        }
    }
}

// Client
class AudioPlayer implements MediaPlayer {
    private MediaAdapter mediaAdapter;
    
    @Override
    public void play(String audioType, String fileName) {
        // Built-in support for mp3
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing mp3 file: " + fileName);
        }
        // Use adapter for other formats
        else if (audioType.equalsIgnoreCase("vlc") || audioType.equalsIgnoreCase("mp4")) {
            mediaAdapter = new MediaAdapter(audioType);
            mediaAdapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid media type: " + audioType);
        }
    }
}
```

### 1.5 Usage Example

```java
public class AdapterExample {
    public static void main(String[] args) {
        AudioPlayer audioPlayer = new AudioPlayer();
        
        audioPlayer.play("mp3", "song.mp3");
        audioPlayer.play("mp4", "video.mp4");
        audioPlayer.play("vlc", "movie.vlc");
        audioPlayer.play("avi", "movie.avi");
    }
}
```

### 1.6 Real-World Example: Payment Gateway Adapter

```java
// Target interface
interface PaymentProcessor {
    void processPayment(double amount, String currency);
}

// Adaptee 1: PayPal
class PayPal {
    public void sendPayment(double amount) {
        System.out.println("PayPal: Sending $" + amount);
    }
}

// Adaptee 2: Stripe
class Stripe {
    public void charge(double amount, String currency) {
        System.out.println("Stripe: Charging " + amount + " " + currency);
    }
}

// Adapter for PayPal
class PayPalAdapter implements PaymentProcessor {
    private PayPal payPal;
    
    public PayPalAdapter(PayPal payPal) {
        this.payPal = payPal;
    }
    
    @Override
    public void processPayment(double amount, String currency) {
        payPal.sendPayment(amount);
    }
}

// Adapter for Stripe
class StripeAdapter implements PaymentProcessor {
    private Stripe stripe;
    
    public StripeAdapter(Stripe stripe) {
        this.stripe = stripe;
    }
    
    @Override
    public void processPayment(double amount, String currency) {
        stripe.charge(amount, currency);
    }
}

// Client
class PaymentService {
    private PaymentProcessor processor;
    
    public PaymentService(PaymentProcessor processor) {
        this.processor = processor;
    }
    
    public void makePayment(double amount, String currency) {
        processor.processPayment(amount, currency);
    }
}

// Usage
public class PaymentAdapterExample {
    public static void main(String[] args) {
        // Use PayPal
        PayPal payPal = new PayPal();
        PaymentProcessor payPalAdapter = new PayPalAdapter(payPal);
        PaymentService service1 = new PaymentService(payPalAdapter);
        service1.makePayment(100.0, "USD");
        
        // Use Stripe
        Stripe stripe = new Stripe();
        PaymentProcessor stripeAdapter = new StripeAdapter(stripe);
        PaymentService service2 = new PaymentService(stripeAdapter);
        service2.makePayment(200.0, "EUR");
    }
}
```

### 1.7 Object Adapter (Composition)

```java
// Target
interface Target {
    void request();
}

// Adaptee
class Adaptee {
    public void specificRequest() {
        System.out.println("Adaptee specific request");
    }
}

// Object Adapter (uses composition)
class ObjectAdapter implements Target {
    private Adaptee adaptee;
    
    public ObjectAdapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }
    
    @Override
    public void request() {
        adaptee.specificRequest();
    }
}
```

---

## 2. Decorator Pattern

### 2.1 Intent

Attach additional responsibilities to an object dynamically. Provides a flexible alternative to subclassing for extending functionality.

### 2.2 When to Use

- Need to add responsibilities to objects dynamically
- Want to add features without modifying existing code
- Subclassing would result in too many classes
- Need to add/remove features at runtime

### 2.3 Scenarios

- **Coffee Shop**: Add toppings to coffee (milk, sugar, whipped cream)
- **Text Processing**: Add formatting (bold, italic, underline)
- **IO Streams**: Add functionality (buffering, compression)
- **Web Framework**: Add middleware (authentication, logging, caching)

### 2.4 Implementation

```java
// Component interface
interface Coffee {
    String getDescription();
    double getCost();
}

// Concrete component
class SimpleCoffee implements Coffee {
    @Override
    public String getDescription() {
        return "Simple Coffee";
    }
    
    @Override
    public double getCost() {
        return 2.0;
    }
}

// Decorator abstract class
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription();
    }
    
    @Override
    public double getCost() {
        return coffee.getCost();
    }
}

// Concrete decorators
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.2;
    }
}

class WhippedCreamDecorator extends CoffeeDecorator {
    public WhippedCreamDecorator(Coffee coffee) {
        super(coffee);
    }
    
    @Override
    public String getDescription() {
        return coffee.getDescription() + ", Whipped Cream";
    }
    
    @Override
    public double getCost() {
        return coffee.getCost() + 0.7;
    }
}
```

### 2.5 Usage Example

```java
public class DecoratorExample {
    public static void main(String[] args) {
        // Simple coffee
        Coffee coffee = new SimpleCoffee();
        System.out.println(coffee.getDescription() + " - $" + coffee.getCost());
        
        // Coffee with milk
        Coffee coffeeWithMilk = new MilkDecorator(new SimpleCoffee());
        System.out.println(coffeeWithMilk.getDescription() + " - $" + coffeeWithMilk.getCost());
        
        // Coffee with milk and sugar
        Coffee coffeeWithMilkAndSugar = new SugarDecorator(new MilkDecorator(new SimpleCoffee()));
        System.out.println(coffeeWithMilkAndSugar.getDescription() + " - $" + coffeeWithMilkAndSugar.getCost());
        
        // Coffee with everything
        Coffee deluxeCoffee = new WhippedCreamDecorator(
            new SugarDecorator(
                new MilkDecorator(new SimpleCoffee())
            )
        );
        System.out.println(deluxeCoffee.getDescription() + " - $" + deluxeCoffee.getCost());
    }
}
```

### 2.6 Real-World Example: Text Processing

```java
// Component
interface Text {
    String format();
}

// Concrete component
class PlainText implements Text {
    private String content;
    
    public PlainText(String content) {
        this.content = content;
    }
    
    @Override
    public String format() {
        return content;
    }
}

// Decorator
abstract class TextDecorator implements Text {
    protected Text text;
    
    public TextDecorator(Text text) {
        this.text = text;
    }
}

// Concrete decorators
class BoldDecorator extends TextDecorator {
    public BoldDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String format() {
        return "<b>" + text.format() + "</b>";
    }
}

class ItalicDecorator extends TextDecorator {
    public ItalicDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String format() {
        return "<i>" + text.format() + "</i>";
    }
}

class UnderlineDecorator extends TextDecorator {
    public UnderlineDecorator(Text text) {
        super(text);
    }
    
    @Override
    public String format() {
        return "<u>" + text.format() + "</u>";
    }
}

// Usage
public class TextDecoratorExample {
    public static void main(String[] args) {
        Text text = new PlainText("Hello World");
        
        // Add formatting
        Text boldText = new BoldDecorator(text);
        System.out.println(boldText.format());
        
        Text boldItalicText = new ItalicDecorator(new BoldDecorator(text));
        System.out.println(boldItalicText.format());
        
        Text fullFormatted = new UnderlineDecorator(
            new ItalicDecorator(
                new BoldDecorator(text)
            )
        );
        System.out.println(fullFormatted.format());
    }
}
```

### 2.7 Java IO Example

```java
import java.io.*;

// Java IO uses Decorator pattern
public class JavaIODecoratorExample {
    public static void main(String[] args) {
        try {
            // Base component: FileOutputStream
            FileOutputStream fileOutputStream = new FileOutputStream("output.txt");
            
            // Decorator 1: BufferedOutputStream (adds buffering)
            BufferedOutputStream bufferedOutputStream = new BufferedOutputStream(fileOutputStream);
            
            // Decorator 2: DataOutputStream (adds data writing methods)
            DataOutputStream dataOutputStream = new DataOutputStream(bufferedOutputStream);
            
            // Use decorated stream
            dataOutputStream.writeUTF("Hello World");
            dataOutputStream.writeInt(42);
            dataOutputStream.writeDouble(3.14);
            
            dataOutputStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

---

## 3. Facade Pattern

### 3.1 Intent

Provide a unified interface to a set of interfaces in a subsystem. Defines a higher-level interface that makes the subsystem easier to use.

### 3.2 When to Use

- Want to provide a simple interface to a complex subsystem
- Need to decouple clients from subsystem components
- Want to layer your system
- Subsystem is getting complex

### 3.3 Scenarios

- **Home Theater System**: Single remote to control TV, DVD, Sound System
- **Order Processing**: Single interface for inventory, payment, shipping
- **API Gateway**: Single entry point for multiple microservices
- **Database Access**: Single interface hiding JDBC complexity

### 3.4 Implementation

```java
// Subsystem classes
class CPU {
    public void freeze() {
        System.out.println("CPU: Freezing...");
    }
    
    public void jump(long position) {
        System.out.println("CPU: Jumping to position " + position);
    }
    
    public void execute() {
        System.out.println("CPU: Executing...");
    }
}

class Memory {
    public void load(long position, byte[] data) {
        System.out.println("Memory: Loading data at position " + position);
    }
}

class HardDrive {
    public byte[] read(long lba, int size) {
        System.out.println("HardDrive: Reading " + size + " bytes from LBA " + lba);
        return new byte[size];
    }
}

// Facade
class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void startComputer() {
        System.out.println("Starting computer...");
        cpu.freeze();
        byte[] bootSector = hardDrive.read(0, 1024);
        memory.load(0, bootSector);
        cpu.jump(0);
        cpu.execute();
        System.out.println("Computer started successfully!");
    }
    
    public void shutdownComputer() {
        System.out.println("Shutting down computer...");
        cpu.freeze();
        System.out.println("Computer shut down successfully!");
    }
}
```

### 3.5 Usage Example

```java
public class FacadeExample {
    public static void main(String[] args) {
        ComputerFacade computer = new ComputerFacade();
        computer.startComputer();
        System.out.println();
        computer.shutdownComputer();
    }
}
```

### 3.6 Real-World Example: Order Processing System

```java
// Subsystem: Inventory
class InventoryService {
    public boolean checkAvailability(String productId, int quantity) {
        System.out.println("Checking availability for product: " + productId);
        // Simulate check
        return true;
    }
    
    public void reserveItems(String productId, int quantity) {
        System.out.println("Reserving " + quantity + " items of product: " + productId);
    }
}

// Subsystem: Payment
class PaymentService {
    public boolean processPayment(double amount, String paymentMethod) {
        System.out.println("Processing payment of $" + amount + " via " + paymentMethod);
        // Simulate payment
        return true;
    }
}

// Subsystem: Shipping
class ShippingService {
    public void shipOrder(String orderId, String address) {
        System.out.println("Shipping order " + orderId + " to " + address);
    }
}

// Subsystem: Notification
class NotificationService {
    public void sendConfirmation(String email, String orderId) {
        System.out.println("Sending confirmation email to " + email + " for order " + orderId);
    }
}

// Facade
class OrderFacade {
    private InventoryService inventoryService;
    private PaymentService paymentService;
    private ShippingService shippingService;
    private NotificationService notificationService;
    
    public OrderFacade() {
        this.inventoryService = new InventoryService();
        this.paymentService = new PaymentService();
        this.shippingService = new ShippingService();
        this.notificationService = new NotificationService();
    }
    
    public boolean placeOrder(String productId, int quantity, double amount, 
                              String paymentMethod, String address, String email) {
        System.out.println("=== Placing Order ===");
        
        // Step 1: Check inventory
        if (!inventoryService.checkAvailability(productId, quantity)) {
            System.out.println("Order failed: Product not available");
            return false;
        }
        
        // Step 2: Reserve items
        inventoryService.reserveItems(productId, quantity);
        
        // Step 3: Process payment
        if (!paymentService.processPayment(amount, paymentMethod)) {
            System.out.println("Order failed: Payment failed");
            return false;
        }
        
        // Step 4: Generate order ID
        String orderId = "ORD-" + System.currentTimeMillis();
        
        // Step 5: Ship order
        shippingService.shipOrder(orderId, address);
        
        // Step 6: Send confirmation
        notificationService.sendConfirmation(email, orderId);
        
        System.out.println("=== Order Placed Successfully ===");
        return true;
    }
}

// Usage
public class OrderFacadeExample {
    public static void main(String[] args) {
        OrderFacade orderFacade = new OrderFacade();
        
        orderFacade.placeOrder(
            "PROD-123",
            2,
            99.99,
            "Credit Card",
            "123 Main St, City",
            "customer@example.com"
        );
    }
}
```

### 3.7 API Gateway Example

```java
// Microservices
class UserService {
    public String getUser(String userId) {
        return "User: " + userId;
    }
}

class ProductService {
    public String getProduct(String productId) {
        return "Product: " + productId;
    }
}

class OrderService {
    public String getOrder(String orderId) {
        return "Order: " + orderId;
    }
}

// API Gateway Facade
class APIGateway {
    private UserService userService;
    private ProductService productService;
    private OrderService orderService;
    
    public APIGateway() {
        this.userService = new UserService();
        this.productService = new ProductService();
        this.orderService = new OrderService();
    }
    
    public String getUserData(String userId) {
        return userService.getUser(userId);
    }
    
    public String getProductData(String productId) {
        return productService.getProduct(productId);
    }
    
    public String getOrderData(String orderId) {
        return orderService.getOrder(orderId);
    }
    
    // Aggregate data from multiple services
    public String getDashboardData(String userId) {
        String user = userService.getUser(userId);
        String orders = orderService.getOrder("ORD-123");
        return "Dashboard: " + user + ", " + orders;
    }
}

// Usage
public class APIGatewayExample {
    public static void main(String[] args) {
        APIGateway gateway = new APIGateway();
        
        System.out.println(gateway.getUserData("USER-123"));
        System.out.println(gateway.getProductData("PROD-456"));
        System.out.println(gateway.getDashboardData("USER-123"));
    }
}
```

---

## Summary: Part 3

### Patterns Covered

1. **Adapter**: Make incompatible interfaces work together
2. **Decorator**: Add responsibilities dynamically
3. **Facade**: Simplify complex subsystem interface

### Key Takeaways

- **Adapter**: Use for integration and legacy code
- **Decorator**: Use for flexible feature addition
- **Facade**: Use to simplify complex systems

### Next Steps

**Part 4** will cover:
- Proxy Pattern
- Bridge Pattern
- Composite Pattern

---

**Master these structural patterns for flexible object composition!**

