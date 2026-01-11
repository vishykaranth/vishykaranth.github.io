# Java Design Patterns: Complete Guide with Examples

## Part 2: Creational Patterns - Builder, Prototype, Object Pool

---

## Table of Contents

1. [Builder Pattern](#1-builder-pattern)
2. [Prototype Pattern](#2-prototype-pattern)
3. [Object Pool Pattern](#3-object-pool-pattern)

---

## 1. Builder Pattern

### 1.1 Intent

Separate the construction of a complex object from its representation, allowing the same construction process to create different representations.

### 1.2 When to Use

- Object has many optional parameters
- Constructor has too many parameters
- Need immutable objects with many fields
- Want to avoid telescoping constructor anti-pattern
- Need step-by-step object construction

### 1.3 Scenarios

- **Database Query Builder**: Build complex SQL queries
- **HTTP Request Builder**: Build HTTP requests with headers, body, etc.
- **Configuration Objects**: Build configuration with many optional settings
- **Email Builder**: Build emails with subject, body, attachments, etc.

### 1.4 Implementation: Classic Builder

```java
// Product class
class Computer {
    private String cpu;
    private String ram;
    private String storage;
    private String graphicsCard;
    private boolean hasBluetooth;
    private boolean hasWiFi;
    
    private Computer(ComputerBuilder builder) {
        this.cpu = builder.cpu;
        this.ram = builder.ram;
        this.storage = builder.storage;
        this.graphicsCard = builder.graphicsCard;
        this.hasBluetooth = builder.hasBluetooth;
        this.hasWiFi = builder.hasWiFi;
    }
    
    // Getters
    public String getCpu() { return cpu; }
    public String getRam() { return ram; }
    public String getStorage() { return storage; }
    public String getGraphicsCard() { return graphicsCard; }
    public boolean hasBluetooth() { return hasBluetooth; }
    public boolean hasWiFi() { return hasWiFi; }
    
    @Override
    public String toString() {
        return "Computer{" +
                "cpu='" + cpu + '\'' +
                ", ram='" + ram + '\'' +
                ", storage='" + storage + '\'' +
                ", graphicsCard='" + graphicsCard + '\'' +
                ", hasBluetooth=" + hasBluetooth +
                ", hasWiFi=" + hasWiFi +
                '}';
    }
    
    // Builder class
    public static class ComputerBuilder {
        // Required parameters
        private String cpu;
        private String ram;
        private String storage;
        
        // Optional parameters
        private String graphicsCard;
        private boolean hasBluetooth = false;
        private boolean hasWiFi = false;
        
        public ComputerBuilder(String cpu, String ram, String storage) {
            this.cpu = cpu;
            this.ram = ram;
            this.storage = storage;
        }
        
        public ComputerBuilder graphicsCard(String graphicsCard) {
            this.graphicsCard = graphicsCard;
            return this;
        }
        
        public ComputerBuilder bluetooth(boolean hasBluetooth) {
            this.hasBluetooth = hasBluetooth;
            return this;
        }
        
        public ComputerBuilder wifi(boolean hasWiFi) {
            this.hasWiFi = hasWiFi;
            return this;
        }
        
        public Computer build() {
            return new Computer(this);
        }
    }
}
```

### 1.5 Usage Example

```java
public class BuilderExample {
    public static void main(String[] args) {
        // Build computer with required and optional parameters
        Computer gamingPC = new Computer.ComputerBuilder(
                "Intel i9", "32GB", "1TB SSD")
                .graphicsCard("RTX 4090")
                .bluetooth(true)
                .wifi(true)
                .build();
        
        System.out.println(gamingPC);
        
        // Build basic computer
        Computer basicPC = new Computer.ComputerBuilder(
                "Intel i5", "8GB", "256GB SSD")
                .build();
        
        System.out.println(basicPC);
    }
}
```

### 1.6 Real-World Example: HTTP Request Builder

```java
import java.util.HashMap;
import java.util.Map;

class HttpRequest {
    private String method;
    private String url;
    private Map<String, String> headers;
    private String body;
    private int timeout;
    
    private HttpRequest(HttpRequestBuilder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.headers = builder.headers;
        this.body = builder.body;
        this.timeout = builder.timeout;
    }
    
    public String getMethod() { return method; }
    public String getUrl() { return url; }
    public Map<String, String> getHeaders() { return headers; }
    public String getBody() { return body; }
    public int getTimeout() { return timeout; }
    
    @Override
    public String toString() {
        return "HttpRequest{" +
                "method='" + method + '\'' +
                ", url='" + url + '\'' +
                ", headers=" + headers +
                ", body='" + body + '\'' +
                ", timeout=" + timeout +
                '}';
    }
    
    public static class HttpRequestBuilder {
        private String method;
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private String body;
        private int timeout = 5000;
        
        public HttpRequestBuilder(String method, String url) {
            this.method = method;
            this.url = url;
        }
        
        public HttpRequestBuilder header(String key, String value) {
            this.headers.put(key, value);
            return this;
        }
        
        public HttpRequestBuilder body(String body) {
            this.body = body;
            return this;
        }
        
        public HttpRequestBuilder timeout(int timeout) {
            this.timeout = timeout;
            return this;
        }
        
        public HttpRequest build() {
            return new HttpRequest(this);
        }
    }
}

// Usage
public class HttpRequestBuilderExample {
    public static void main(String[] args) {
        HttpRequest request = new HttpRequest.HttpRequestBuilder("POST", "https://api.example.com/users")
                .header("Content-Type", "application/json")
                .header("Authorization", "Bearer token123")
                .body("{\"name\":\"John\",\"email\":\"john@example.com\"}")
                .timeout(10000)
                .build();
        
        System.out.println(request);
    }
}
```

### 1.7 Fluent Builder with Validation

```java
class User {
    private String firstName;
    private String lastName;
    private String email;
    private int age;
    
    private User(UserBuilder builder) {
        this.firstName = builder.firstName;
        this.lastName = builder.lastName;
        this.email = builder.email;
        this.age = builder.age;
    }
    
    // Getters...
    
    public static class UserBuilder {
        private String firstName;
        private String lastName;
        private String email;
        private int age;
        
        public UserBuilder firstName(String firstName) {
            this.firstName = firstName;
            return this;
        }
        
        public UserBuilder lastName(String lastName) {
            this.lastName = lastName;
            return this;
        }
        
        public UserBuilder email(String email) {
            this.email = email;
            return this;
        }
        
        public UserBuilder age(int age) {
            this.age = age;
            return this;
        }
        
        public User build() {
            // Validation
            if (firstName == null || firstName.isEmpty()) {
                throw new IllegalArgumentException("First name is required");
            }
            if (lastName == null || lastName.isEmpty()) {
                throw new IllegalArgumentException("Last name is required");
            }
            if (email == null || !email.contains("@")) {
                throw new IllegalArgumentException("Valid email is required");
            }
            if (age < 0 || age > 150) {
                throw new IllegalArgumentException("Age must be between 0 and 150");
            }
            
            return new User(this);
        }
    }
}
```

---

## 2. Prototype Pattern

### 2.1 Intent

Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

### 2.2 When to Use

- Object creation is expensive
- Want to avoid subclasses of creator classes
- Objects have many shared configurations
- Need to create objects at runtime
- Want to reduce initialization cost

### 2.3 Scenarios

- **Game Development**: Clone game objects (characters, weapons)
- **Database Operations**: Clone query templates
- **Configuration Objects**: Clone and modify configurations
- **Document Processing**: Clone document templates

### 2.4 Implementation: Shallow Copy

```java
import java.util.ArrayList;
import java.util.List;

// Prototype interface
interface Prototype {
    Prototype clone();
}

// Concrete prototype
class Document implements Prototype {
    private String title;
    private String content;
    private List<String> tags;
    
    public Document(String title, String content) {
        this.title = title;
        this.content = content;
        this.tags = new ArrayList<>();
    }
    
    // Copy constructor
    public Document(Document document) {
        this.title = document.title;
        this.content = document.content;
        this.tags = new ArrayList<>(document.tags); // Shallow copy
    }
    
    public void addTag(String tag) {
        tags.add(tag);
    }
    
    public void setTitle(String title) {
        this.title = title;
    }
    
    public void setContent(String content) {
        this.content = content;
    }
    
    @Override
    public Prototype clone() {
        return new Document(this);
    }
    
    @Override
    public String toString() {
        return "Document{" +
                "title='" + title + '\'' +
                ", content='" + content + '\'' +
                ", tags=" + tags +
                '}';
    }
}
```

### 2.5 Usage Example

```java
public class PrototypeExample {
    public static void main(String[] args) {
        // Create prototype
        Document original = new Document("Template", "This is a template document");
        original.addTag("template");
        original.addTag("document");
        
        System.out.println("Original: " + original);
        
        // Clone prototype
        Document clone1 = (Document) original.clone();
        clone1.setTitle("Report 1");
        clone1.setContent("This is report 1");
        clone1.addTag("report");
        
        System.out.println("Clone 1: " + clone1);
        
        // Clone again
        Document clone2 = (Document) original.clone();
        clone2.setTitle("Report 2");
        clone2.setContent("This is report 2");
        clone2.addTag("report");
        
        System.out.println("Clone 2: " + clone2);
        System.out.println("Original unchanged: " + original);
    }
}
```

### 2.6 Real-World Example: Shape Prototype

```java
// Prototype interface
interface Shape extends Cloneable {
    Shape clone();
    void draw();
    void setColor(String color);
}

// Concrete prototypes
class Circle implements Shape {
    private int radius;
    private String color;
    
    public Circle(int radius) {
        this.radius = radius;
        this.color = "Red";
    }
    
    // Copy constructor
    public Circle(Circle circle) {
        this.radius = circle.radius;
        this.color = circle.color;
    }
    
    @Override
    public Shape clone() {
        return new Circle(this);
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing Circle with radius " + radius + " and color " + color);
    }
    
    @Override
    public void setColor(String color) {
        this.color = color;
    }
}

class Rectangle implements Shape {
    private int width;
    private int height;
    private String color;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
        this.color = "Blue";
    }
    
    public Rectangle(Rectangle rectangle) {
        this.width = rectangle.width;
        this.height = rectangle.height;
        this.color = rectangle.color;
    }
    
    @Override
    public Shape clone() {
        return new Rectangle(this);
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing Rectangle " + width + "x" + height + " with color " + color);
    }
    
    @Override
    public void setColor(String color) {
        this.color = color;
    }
}

// Prototype registry
class ShapeRegistry {
    private static final Map<String, Shape> shapes = new HashMap<>();
    
    static {
        shapes.put("circle", new Circle(10));
        shapes.put("rectangle", new Rectangle(20, 30));
    }
    
    public static Shape getShape(String type) {
        return shapes.get(type).clone();
    }
}

// Usage
public class ShapePrototypeExample {
    public static void main(String[] args) {
        // Get shapes from registry
        Shape circle1 = ShapeRegistry.getShape("circle");
        circle1.setColor("Green");
        circle1.draw();
        
        Shape circle2 = ShapeRegistry.getShape("circle");
        circle2.setColor("Yellow");
        circle2.draw();
        
        Shape rectangle = ShapeRegistry.getShape("rectangle");
        rectangle.setColor("Purple");
        rectangle.draw();
    }
}
```

### 2.7 Deep Copy Implementation

```java
import java.io.*;

class DeepCopyPrototype implements Serializable {
    private String name;
    private List<String> items;
    
    public DeepCopyPrototype(String name) {
        this.name = name;
        this.items = new ArrayList<>();
    }
    
    public void addItem(String item) {
        items.add(item);
    }
    
    // Deep copy using serialization
    public DeepCopyPrototype deepCopy() {
        try {
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            ObjectOutputStream oos = new ObjectOutputStream(baos);
            oos.writeObject(this);
            
            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bais);
            
            return (DeepCopyPrototype) ois.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException("Deep copy failed", e);
        }
    }
    
    @Override
    public String toString() {
        return "DeepCopyPrototype{" +
                "name='" + name + '\'' +
                ", items=" + items +
                '}';
    }
}
```

---

## 3. Object Pool Pattern

### 3.1 Intent

Reuse objects that are expensive to create, rather than creating new ones repeatedly.

### 3.2 When to Use

- Object creation is expensive
- Objects are frequently created and destroyed
- Limited number of objects needed
- Objects have similar initialization
- Need to control resource usage

### 3.3 Scenarios

- **Database Connections**: Pool of database connections
- **Thread Pool**: Reuse threads instead of creating new ones
- **HTTP Client Pool**: Reuse HTTP clients
- **Memory-Intensive Objects**: Pool large objects

### 3.4 Implementation

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.TimeUnit;

// Pooled object
class DatabaseConnection {
    private String connectionId;
    private boolean inUse;
    
    public DatabaseConnection(String connectionId) {
        this.connectionId = connectionId;
        this.inUse = false;
    }
    
    public void connect() {
        System.out.println("Connecting to database: " + connectionId);
    }
    
    public void executeQuery(String query) {
        System.out.println("Executing query on " + connectionId + ": " + query);
    }
    
    public void disconnect() {
        System.out.println("Disconnecting from database: " + connectionId);
    }
    
    public boolean isInUse() {
        return inUse;
    }
    
    public void setInUse(boolean inUse) {
        this.inUse = inUse;
    }
    
    public String getConnectionId() {
        return connectionId;
    }
}

// Object pool
class ConnectionPool {
    private static ConnectionPool instance;
    private BlockingQueue<DatabaseConnection> availableConnections;
    private BlockingQueue<DatabaseConnection> usedConnections;
    private int maxSize;
    
    private ConnectionPool(int maxSize) {
        this.maxSize = maxSize;
        this.availableConnections = new LinkedBlockingQueue<>();
        this.usedConnections = new LinkedBlockingQueue<>();
        initializePool();
    }
    
    public static synchronized ConnectionPool getInstance(int maxSize) {
        if (instance == null) {
            instance = new ConnectionPool(maxSize);
        }
        return instance;
    }
    
    private void initializePool() {
        for (int i = 1; i <= maxSize; i++) {
            DatabaseConnection connection = new DatabaseConnection("Connection-" + i);
            availableConnections.offer(connection);
        }
    }
    
    public DatabaseConnection acquireConnection() throws InterruptedException {
        DatabaseConnection connection = availableConnections.poll(5, TimeUnit.SECONDS);
        if (connection != null) {
            connection.setInUse(true);
            usedConnections.offer(connection);
            connection.connect();
        }
        return connection;
    }
    
    public void releaseConnection(DatabaseConnection connection) {
        if (connection != null && connection.isInUse()) {
            connection.disconnect();
            connection.setInUse(false);
            usedConnections.remove(connection);
            availableConnections.offer(connection);
        }
    }
    
    public int getAvailableConnections() {
        return availableConnections.size();
    }
    
    public int getUsedConnections() {
        return usedConnections.size();
    }
}
```

### 3.5 Usage Example

```java
public class ObjectPoolExample {
    public static void main(String[] args) {
        ConnectionPool pool = ConnectionPool.getInstance(3);
        
        try {
            // Acquire connections
            DatabaseConnection conn1 = pool.acquireConnection();
            DatabaseConnection conn2 = pool.acquireConnection();
            DatabaseConnection conn3 = pool.acquireConnection();
            
            // Use connections
            if (conn1 != null) {
                conn1.executeQuery("SELECT * FROM users");
            }
            if (conn2 != null) {
                conn2.executeQuery("SELECT * FROM products");
            }
            
            System.out.println("Available: " + pool.getAvailableConnections());
            System.out.println("Used: " + pool.getUsedConnections());
            
            // Release connections
            pool.releaseConnection(conn1);
            pool.releaseConnection(conn2);
            
            System.out.println("After release - Available: " + pool.getAvailableConnections());
            System.out.println("After release - Used: " + pool.getUsedConnections());
            
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

### 3.6 Real-World Example: Thread Pool

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

class Task implements Runnable {
    private int taskId;
    
    public Task(int taskId) {
        this.taskId = taskId;
    }
    
    @Override
    public void run() {
        System.out.println("Task " + taskId + " is running on " + Thread.currentThread().getName());
        try {
            Thread.sleep(1000); // Simulate work
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        System.out.println("Task " + taskId + " completed");
    }
}

public class ThreadPoolExample {
    public static void main(String[] args) {
        // Create thread pool (object pool for threads)
        ExecutorService executor = Executors.newFixedThreadPool(3);
        
        // Submit tasks
        for (int i = 1; i <= 10; i++) {
            executor.submit(new Task(i));
        }
        
        // Shutdown pool
        executor.shutdown();
        try {
            if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
                executor.shutdownNow();
            }
        } catch (InterruptedException e) {
            executor.shutdownNow();
        }
    }
}
```

---

## Summary: Part 2

### Patterns Covered

1. **Builder**: Step-by-step object construction
2. **Prototype**: Clone objects to avoid expensive creation
3. **Object Pool**: Reuse expensive objects

### Key Takeaways

- **Builder**: Use for objects with many optional parameters
- **Prototype**: Use when object creation is expensive
- **Object Pool**: Use for frequently created/destroyed objects

### Next Steps

**Part 3** will cover:
- Adapter Pattern
- Decorator Pattern
- Facade Pattern

---

**Master these creational patterns for efficient object creation!**

