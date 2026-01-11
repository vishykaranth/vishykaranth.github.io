# Java Design Patterns: Complete Guide with Examples

## Part 1: Creational Patterns - Singleton, Factory, Abstract Factory

---

## Table of Contents

1. [Design Patterns Overview](#1-design-patterns-overview)
2. [Singleton Pattern](#2-singleton-pattern)
3. [Factory Method Pattern](#3-factory-method-pattern)
4. [Abstract Factory Pattern](#4-abstract-factory-pattern)

---

## 1. Design Patterns Overview

### 1.1 What are Design Patterns?

**Design Patterns** are reusable solutions to common problems in software design. They provide templates for solving recurring design problems.

### 1.2 Categories

1. **Creational Patterns**: Object creation mechanisms
2. **Structural Patterns**: Object composition and relationships
3. **Behavioral Patterns**: Communication between objects
4. **Concurrency Patterns**: Multi-threaded programming

---

## 2. Singleton Pattern

### 2.1 Intent

Ensure a class has only one instance and provide a global point of access to it.

### 2.2 When to Use

- Need exactly one instance of a class
- Global access point required
- Lazy initialization needed
- Resource-intensive object creation

### 2.3 Scenarios

- **Database Connection**: Single connection pool
- **Logger**: Single logging instance
- **Configuration Manager**: Single config instance
- **Cache Manager**: Single cache instance

### 2.4 Implementation: Eager Initialization

```java
public class EagerSingleton {
    // Private static instance created at class loading time
    private static final EagerSingleton instance = new EagerSingleton();
    
    // Private constructor to prevent instantiation
    private EagerSingleton() {
        // Prevent reflection
        if (instance != null) {
            throw new RuntimeException("Use getInstance() to get the instance");
        }
    }
    
    // Public static method to get instance
    public static EagerSingleton getInstance() {
        return instance;
    }
    
    public void doSomething() {
        System.out.println("Eager Singleton doing something");
    }
}
```

### 2.5 Implementation: Lazy Initialization (Thread-Safe)

```java
public class LazySingleton {
    // Private static instance
    private static LazySingleton instance;
    
    private LazySingleton() {
    }
    
    // Synchronized method for thread safety
    public static synchronized LazySingleton getInstance() {
        if (instance == null) {
            instance = new LazySingleton();
        }
        return instance;
    }
    
    public void doSomething() {
        System.out.println("Lazy Singleton doing something");
    }
}
```

### 2.6 Implementation: Double-Checked Locking

```java
public class DoubleCheckedLockingSingleton {
    // Volatile ensures visibility across threads
    private static volatile DoubleCheckedLockingSingleton instance;
    
    private DoubleCheckedLockingSingleton() {
    }
    
    public static DoubleCheckedLockingSingleton getInstance() {
        // First check (no synchronization)
        if (instance == null) {
            // Synchronize only when instance is null
            synchronized (DoubleCheckedLockingSingleton.class) {
                // Second check (inside synchronized block)
                if (instance == null) {
                    instance = new DoubleCheckedLockingSingleton();
                }
            }
        }
        return instance;
    }
    
    public void doSomething() {
        System.out.println("Double-Checked Locking Singleton");
    }
}
```

### 2.7 Implementation: Bill Pugh Solution (Recommended)

```java
public class BillPughSingleton {
    private BillPughSingleton() {
    }
    
    // Static inner class - loaded only when getInstance() is called
    private static class SingletonHelper {
        private static final BillPughSingleton INSTANCE = new BillPughSingleton();
    }
    
    public static BillPughSingleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
    
    public void doSomething() {
        System.out.println("Bill Pugh Singleton");
    }
}
```

### 2.8 Real-World Example: Logger

```java
public class Logger {
    private static Logger instance;
    private String logFile = "application.log";
    
    private Logger() {
    }
    
    public static synchronized Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }
    
    public void log(String message) {
        // Write to log file
        System.out.println("[" + new Date() + "] " + message);
    }
}

// Usage
public class LoggerExample {
    public static void main(String[] args) {
        Logger logger = Logger.getInstance();
        logger.log("Application started");
        
        // Same instance
        Logger logger2 = Logger.getInstance();
        logger2.log("Another log message");
        
        System.out.println(logger == logger2); // true
    }
}
```

---

## 3. Factory Method Pattern

### 3.1 Intent

Define an interface for creating objects, but let subclasses decide which class to instantiate.

### 3.2 When to Use

- Don't know exact types of objects at compile time
- Want to delegate object creation to subclasses
- Need to add new product types without modifying existing code
- Class can't anticipate the class of objects it must create

### 3.3 Scenarios

- **Payment Processing**: Different payment methods (Credit Card, PayPal, Bank Transfer)
- **Database Connections**: Different database types (MySQL, PostgreSQL, MongoDB)
- **UI Components**: Different platforms (Windows, Mac, Linux)
- **Notification Systems**: Different channels (Email, SMS, Push)

### 3.4 Implementation

```java
// Product interface
interface Notification {
    void send(String message);
}

// Concrete products
class EmailNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

class SMSNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

class PushNotification implements Notification {
    @Override
    public void send(String message) {
        System.out.println("Sending push notification: " + message);
    }
}

// Creator abstract class
abstract class NotificationFactory {
    // Factory method
    public abstract Notification createNotification();
    
    // Template method
    public void sendNotification(String message) {
        Notification notification = createNotification();
        notification.send(message);
    }
}

// Concrete creators
class EmailNotificationFactory extends NotificationFactory {
    @Override
    public Notification createNotification() {
        return new EmailNotification();
    }
}

class SMSNotificationFactory extends NotificationFactory {
    @Override
    public Notification createNotification() {
        return new SMSNotification();
    }
}

class PushNotificationFactory extends NotificationFactory {
    @Override
    public Notification createNotification() {
        return new PushNotification();
    }
}
```

### 3.5 Usage Example

```java
public class FactoryMethodExample {
    public static void main(String[] args) {
        // Create factory
        NotificationFactory emailFactory = new EmailNotificationFactory();
        emailFactory.sendNotification("Hello via Email");
        
        NotificationFactory smsFactory = new SMSNotificationFactory();
        smsFactory.sendNotification("Hello via SMS");
        
        NotificationFactory pushFactory = new PushNotificationFactory();
        pushFactory.sendNotification("Hello via Push");
    }
}
```

### 3.6 Real-World Example: Database Connection Factory

```java
// Product interface
interface DatabaseConnection {
    void connect();
    void executeQuery(String query);
    void disconnect();
}

// Concrete products
class MySQLConnection implements DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to MySQL database");
    }
    
    @Override
    public void executeQuery(String query) {
        System.out.println("Executing MySQL query: " + query);
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from MySQL");
    }
}

class PostgreSQLConnection implements DatabaseConnection {
    @Override
    public void connect() {
        System.out.println("Connecting to PostgreSQL database");
    }
    
    @Override
    public void executeQuery(String query) {
        System.out.println("Executing PostgreSQL query: " + query);
    }
    
    @Override
    public void disconnect() {
        System.out.println("Disconnecting from PostgreSQL");
    }
}

// Factory
abstract class DatabaseFactory {
    public abstract DatabaseConnection createConnection();
    
    public void executeQuery(String query) {
        DatabaseConnection connection = createConnection();
        connection.connect();
        connection.executeQuery(query);
        connection.disconnect();
    }
}

class MySQLFactory extends DatabaseFactory {
    @Override
    public DatabaseConnection createConnection() {
        return new MySQLConnection();
    }
}

class PostgreSQLFactory extends DatabaseFactory {
    @Override
    public DatabaseConnection createConnection() {
        return new PostgreSQLConnection();
    }
}

// Usage
public class DatabaseFactoryExample {
    public static void main(String[] args) {
        DatabaseFactory mysqlFactory = new MySQLFactory();
        mysqlFactory.executeQuery("SELECT * FROM users");
        
        DatabaseFactory postgresFactory = new PostgreSQLFactory();
        postgresFactory.executeQuery("SELECT * FROM products");
    }
}
```

---

## 4. Abstract Factory Pattern

### 4.1 Intent

Provide an interface for creating families of related or dependent objects without specifying their concrete classes.

### 4.2 When to Use

- System should be independent of how products are created
- System should work with multiple product families
- Products in a family are designed to work together
- Need to enforce constraints on product combinations

### 4.3 Scenarios

- **UI Framework**: Different themes (Dark, Light) with matching components
- **Cross-Platform**: Different OS (Windows, Mac, Linux) with platform-specific components
- **Database Abstraction**: Different databases with matching connection, query, transaction objects
- **Game Development**: Different game themes with matching characters, weapons, levels

### 4.4 Implementation

```java
// Abstract products
interface Button {
    void render();
    void onClick();
}

interface Checkbox {
    void render();
    void onCheck();
}

// Concrete products - Windows family
class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows button");
    }
    
    @Override
    public void onClick() {
        System.out.println("Windows button clicked");
    }
}

class WindowsCheckbox implements Checkbox {
    @Override
    public void render() {
        System.out.println("Rendering Windows checkbox");
    }
    
    @Override
    public void onCheck() {
        System.out.println("Windows checkbox checked");
    }
}

// Concrete products - Mac family
class MacButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Mac button");
    }
    
    @Override
    public void onClick() {
        System.out.println("Mac button clicked");
    }
}

class MacCheckbox implements Checkbox {
    @Override
    public void render() {
        System.out.println("Rendering Mac checkbox");
    }
    
    @Override
    public void onCheck() {
        System.out.println("Mac checkbox checked");
    }
}

// Abstract factory
interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete factories
class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Client code
class Application {
    private Button button;
    private Checkbox checkbox;
    
    public Application(GUIFactory factory) {
        this.button = factory.createButton();
        this.checkbox = factory.createCheckbox();
    }
    
    public void render() {
        button.render();
        checkbox.render();
    }
    
    public void interact() {
        button.onClick();
        checkbox.onCheck();
    }
}
```

### 4.5 Usage Example

```java
public class AbstractFactoryExample {
    public static void main(String[] args) {
        // Create Windows application
        GUIFactory windowsFactory = new WindowsFactory();
        Application windowsApp = new Application(windowsFactory);
        windowsApp.render();
        windowsApp.interact();
        
        System.out.println("---");
        
        // Create Mac application
        GUIFactory macFactory = new MacFactory();
        Application macApp = new Application(macFactory);
        macApp.render();
        macApp.interact();
    }
}
```

### 4.6 Real-World Example: Database Factory

```java
// Abstract products
interface Connection {
    void connect();
}

interface Statement {
    void execute(String sql);
}

interface ResultSet {
    void fetch();
}

// MySQL family
class MySQLConnection implements Connection {
    @Override
    public void connect() {
        System.out.println("MySQL connection established");
    }
}

class MySQLStatement implements Statement {
    @Override
    public void execute(String sql) {
        System.out.println("MySQL executing: " + sql);
    }
}

class MySQLResultSet implements ResultSet {
    @Override
    public void fetch() {
        System.out.println("MySQL fetching results");
    }
}

// PostgreSQL family
class PostgreSQLConnection implements Connection {
    @Override
    public void connect() {
        System.out.println("PostgreSQL connection established");
    }
}

class PostgreSQLStatement implements Statement {
    @Override
    public void execute(String sql) {
        System.out.println("PostgreSQL executing: " + sql);
    }
}

class PostgreSQLResultSet implements ResultSet {
    @Override
    public void fetch() {
        System.out.println("PostgreSQL fetching results");
    }
}

// Abstract factory
interface DatabaseFactory {
    Connection createConnection();
    Statement createStatement();
    ResultSet createResultSet();
}

// Concrete factories
class MySQLFactory implements DatabaseFactory {
    @Override
    public Connection createConnection() {
        return new MySQLConnection();
    }
    
    @Override
    public Statement createStatement() {
        return new MySQLStatement();
    }
    
    @Override
    public ResultSet createResultSet() {
        return new MySQLResultSet();
    }
}

class PostgreSQLFactory implements DatabaseFactory {
    @Override
    public Connection createConnection() {
        return new PostgreSQLConnection();
    }
    
    @Override
    public Statement createStatement() {
        return new PostgreSQLStatement();
    }
    
    @Override
    public ResultSet createResultSet() {
        return new PostgreSQLResultSet();
    }
}

// Client
class DatabaseClient {
    private Connection connection;
    private Statement statement;
    private ResultSet resultSet;
    
    public DatabaseClient(DatabaseFactory factory) {
        this.connection = factory.createConnection();
        this.statement = factory.createStatement();
        this.resultSet = factory.createResultSet();
    }
    
    public void executeQuery(String sql) {
        connection.connect();
        statement.execute(sql);
        resultSet.fetch();
    }
}

// Usage
public class DatabaseFactoryExample {
    public static void main(String[] args) {
        DatabaseFactory mysqlFactory = new MySQLFactory();
        DatabaseClient mysqlClient = new DatabaseClient(mysqlFactory);
        mysqlClient.executeQuery("SELECT * FROM users");
        
        System.out.println("---");
        
        DatabaseFactory postgresFactory = new PostgreSQLFactory();
        DatabaseClient postgresClient = new DatabaseClient(postgresFactory);
        postgresClient.executeQuery("SELECT * FROM products");
    }
}
```

---

## Summary: Part 1

### Patterns Covered

1. **Singleton**: Single instance, global access
2. **Factory Method**: Delegate creation to subclasses
3. **Abstract Factory**: Create families of related objects

### Key Takeaways

- **Singleton**: Use for shared resources (Logger, Config)
- **Factory Method**: Use when object type varies
- **Abstract Factory**: Use for related object families

### Next Steps

**Part 2** will cover:
- Builder Pattern
- Prototype Pattern
- Object Pool Pattern

---

**Master these creational patterns to create objects effectively!**

