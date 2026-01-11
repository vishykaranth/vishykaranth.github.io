# Java Design Patterns: Complete Guide with Examples

## Part 4: Structural Patterns - Proxy, Bridge, Composite

---

## Table of Contents

1. [Proxy Pattern](#1-proxy-pattern)
2. [Bridge Pattern](#2-bridge-pattern)
3. [Composite Pattern](#3-composite-pattern)

---

## 1. Proxy Pattern

### 1.1 Intent

Provide a surrogate or placeholder for another object to control access to it.

### 1.2 When to Use

- Need to control access to an object
- Want to add functionality before/after object access
- Object creation is expensive (lazy loading)
- Need to add security/permission checks
- Want to add logging/caching

### 1.3 Scenarios

- **Virtual Proxy**: Lazy loading of expensive objects
- **Protection Proxy**: Access control and security
- **Remote Proxy**: Represent remote objects
- **Caching Proxy**: Cache expensive operations
- **Logging Proxy**: Add logging to method calls

### 1.4 Implementation: Virtual Proxy

```java
// Subject interface
interface Image {
    void display();
}

// Real subject
class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading image from disk: " + filename);
        // Simulate expensive operation
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
    
    @Override
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}

// Proxy
class ImageProxy implements Image {
    private RealImage realImage;
    private String filename;
    
    public ImageProxy(String filename) {
        this.filename = filename;
    }
    
    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename); // Lazy loading
        }
        realImage.display();
    }
}
```

### 1.5 Usage Example

```java
public class ProxyExample {
    public static void main(String[] args) {
        // Image is not loaded until display() is called
        Image image = new ImageProxy("photo.jpg");
        System.out.println("Image proxy created (image not loaded yet)");
        
        // Image is loaded only when needed
        image.display();
        System.out.println();
        
        // Image already loaded, no need to load again
        image.display();
    }
}
```

### 1.6 Real-World Example: Protection Proxy

```java
// Subject
interface BankAccount {
    void deposit(double amount);
    void withdraw(double amount);
    double getBalance();
}

// Real subject
class RealBankAccount implements BankAccount {
    private double balance;
    
    public RealBankAccount(double initialBalance) {
        this.balance = initialBalance;
    }
    
    @Override
    public void deposit(double amount) {
        balance += amount;
        System.out.println("Deposited: $" + amount);
    }
    
    @Override
    public void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
            System.out.println("Withdrawn: $" + amount);
        } else {
            System.out.println("Insufficient funds");
        }
    }
    
    @Override
    public double getBalance() {
        return balance;
    }
}

// Protection proxy
class BankAccountProxy implements BankAccount {
    private RealBankAccount realAccount;
    private String userRole;
    
    public BankAccountProxy(String userRole) {
        this.userRole = userRole;
        this.realAccount = new RealBankAccount(1000.0);
    }
    
    private boolean hasPermission(String operation) {
        if ("admin".equals(userRole)) {
            return true;
        }
        return "deposit".equals(operation) || "balance".equals(operation);
    }
    
    @Override
    public void deposit(double amount) {
        if (hasPermission("deposit")) {
            realAccount.deposit(amount);
        } else {
            System.out.println("Access denied: Cannot deposit");
        }
    }
    
    @Override
    public void withdraw(double amount) {
        if (hasPermission("withdraw")) {
            realAccount.withdraw(amount);
        } else {
            System.out.println("Access denied: Cannot withdraw");
        }
    }
    
    @Override
    public double getBalance() {
        if (hasPermission("balance")) {
            return realAccount.getBalance();
        } else {
            System.out.println("Access denied: Cannot view balance");
            return 0;
        }
    }
}

// Usage
public class ProtectionProxyExample {
    public static void main(String[] args) {
        // Admin has full access
        BankAccount adminAccount = new BankAccountProxy("admin");
        adminAccount.deposit(100);
        adminAccount.withdraw(50);
        System.out.println("Balance: $" + adminAccount.getBalance());
        
        System.out.println();
        
        // User has limited access
        BankAccount userAccount = new BankAccountProxy("user");
        userAccount.deposit(100);
        userAccount.withdraw(50); // Denied
        System.out.println("Balance: $" + userAccount.getBalance());
    }
}
```

### 1.7 Caching Proxy Example

```java
import java.util.HashMap;
import java.util.Map;

// Subject
interface DataService {
    String fetchData(String key);
}

// Real subject
class RealDataService implements DataService {
    @Override
    public String fetchData(String key) {
        System.out.println("Fetching data from database for key: " + key);
        // Simulate expensive database call
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return "Data for " + key;
    }
}

// Caching proxy
class CachingProxy implements DataService {
    private RealDataService realService;
    private Map<String, String> cache;
    
    public CachingProxy() {
        this.realService = new RealDataService();
        this.cache = new HashMap<>();
    }
    
    @Override
    public String fetchData(String key) {
        // Check cache first
        if (cache.containsKey(key)) {
            System.out.println("Returning cached data for key: " + key);
            return cache.get(key);
        }
        
        // Fetch from real service
        String data = realService.fetchData(key);
        cache.put(key, data); // Cache the result
        return data;
    }
    
    public void clearCache() {
        cache.clear();
        System.out.println("Cache cleared");
    }
}

// Usage
public class CachingProxyExample {
    public static void main(String[] args) {
        CachingProxy proxy = new CachingProxy();
        
        // First call - fetches from database
        System.out.println(proxy.fetchData("key1"));
        System.out.println();
        
        // Second call - returns from cache
        System.out.println(proxy.fetchData("key1"));
    }
}
```

---

## 2. Bridge Pattern

### 2.1 Intent

Decouple an abstraction from its implementation so that the two can vary independently.

### 2.2 When to Use

- Want to avoid permanent binding between abstraction and implementation
- Need to switch implementations at runtime
- Changes in implementation should not affect clients
- Want to hide implementation details from clients
- Have a proliferation of classes from combination of interfaces and implementations

### 2.3 Scenarios

- **Device and Remote Control**: Different devices with different remote controls
- **Shape and Drawing**: Different shapes with different drawing APIs
- **Database Drivers**: Different databases with different connection implementations
- **Platform Independence**: Different platforms with different implementations

### 2.4 Implementation

```java
// Implementation interface
interface DrawingAPI {
    void drawCircle(double x, double y, double radius);
    void drawRectangle(double x, double y, double width, double height);
}

// Concrete implementation 1
class VectorDrawingAPI implements DrawingAPI {
    @Override
    public void drawCircle(double x, double y, double radius) {
        System.out.println("Vector: Drawing circle at (" + x + "," + y + ") with radius " + radius);
    }
    
    @Override
    public void drawRectangle(double x, double y, double width, double height) {
        System.out.println("Vector: Drawing rectangle at (" + x + "," + y + ") with size " + width + "x" + height);
    }
}

// Concrete implementation 2
class RasterDrawingAPI implements DrawingAPI {
    @Override
    public void drawCircle(double x, double y, double radius) {
        System.out.println("Raster: Drawing circle at (" + x + "," + y + ") with radius " + radius);
    }
    
    @Override
    public void drawRectangle(double x, double y, double width, double height) {
        System.out.println("Raster: Drawing rectangle at (" + x + "," + y + ") with size " + width + "x" + height);
    }
}

// Abstraction
abstract class Shape {
    protected DrawingAPI drawingAPI;
    
    protected Shape(DrawingAPI drawingAPI) {
        this.drawingAPI = drawingAPI;
    }
    
    public abstract void draw();
    public abstract void resize(double factor);
}

// Refined abstraction 1
class Circle extends Shape {
    private double x, y, radius;
    
    public Circle(double x, double y, double radius, DrawingAPI drawingAPI) {
        super(drawingAPI);
        this.x = x;
        this.y = y;
        this.radius = radius;
    }
    
    @Override
    public void draw() {
        drawingAPI.drawCircle(x, y, radius);
    }
    
    @Override
    public void resize(double factor) {
        radius *= factor;
    }
}

// Refined abstraction 2
class Rectangle extends Shape {
    private double x, y, width, height;
    
    public Rectangle(double x, double y, double width, double height, DrawingAPI drawingAPI) {
        super(drawingAPI);
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }
    
    @Override
    public void draw() {
        drawingAPI.drawRectangle(x, y, width, height);
    }
    
    @Override
    public void resize(double factor) {
        width *= factor;
        height *= factor;
    }
}
```

### 2.5 Usage Example

```java
public class BridgeExample {
    public static void main(String[] args) {
        DrawingAPI vectorAPI = new VectorDrawingAPI();
        DrawingAPI rasterAPI = new RasterDrawingAPI();
        
        // Circle with vector drawing
        Shape circle1 = new Circle(1, 2, 3, vectorAPI);
        circle1.draw();
        
        // Circle with raster drawing
        Shape circle2 = new Circle(5, 7, 10, rasterAPI);
        circle2.draw();
        
        // Rectangle with vector drawing
        Shape rectangle1 = new Rectangle(1, 1, 5, 3, vectorAPI);
        rectangle1.draw();
        
        // Rectangle with raster drawing
        Shape rectangle2 = new Rectangle(2, 2, 4, 6, rasterAPI);
        rectangle2.draw();
    }
}
```

### 2.6 Real-World Example: Device and Remote Control

```java
// Implementation interface
interface Device {
    void turnOn();
    void turnOff();
    void setChannel(int channel);
    void setVolume(int volume);
    boolean isEnabled();
    int getChannel();
    int getVolume();
}

// Concrete implementation 1
class TV implements Device {
    private boolean on = false;
    private int volume = 30;
    private int channel = 1;
    
    @Override
    public void turnOn() {
        on = true;
        System.out.println("TV is now ON");
    }
    
    @Override
    public void turnOff() {
        on = false;
        System.out.println("TV is now OFF");
    }
    
    @Override
    public void setChannel(int channel) {
        this.channel = channel;
        System.out.println("TV channel set to " + channel);
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("TV volume set to " + volume);
    }
    
    @Override
    public boolean isEnabled() {
        return on;
    }
    
    @Override
    public int getChannel() {
        return channel;
    }
    
    @Override
    public int getVolume() {
        return volume;
    }
}

// Concrete implementation 2
class Radio implements Device {
    private boolean on = false;
    private int volume = 50;
    private int channel = 1;
    
    @Override
    public void turnOn() {
        on = true;
        System.out.println("Radio is now ON");
    }
    
    @Override
    public void turnOff() {
        on = false;
        System.out.println("Radio is now OFF");
    }
    
    @Override
    public void setChannel(int channel) {
        this.channel = channel;
        System.out.println("Radio channel set to " + channel);
    }
    
    @Override
    public void setVolume(int volume) {
        this.volume = volume;
        System.out.println("Radio volume set to " + volume);
    }
    
    @Override
    public boolean isEnabled() {
        return on;
    }
    
    @Override
    public int getChannel() {
        return channel;
    }
    
    @Override
    public int getVolume() {
        return volume;
    }
}

// Abstraction
class RemoteControl {
    protected Device device;
    
    public RemoteControl(Device device) {
        this.device = device;
    }
    
    public void togglePower() {
        if (device.isEnabled()) {
            device.turnOff();
        } else {
            device.turnOn();
        }
    }
    
    public void volumeDown() {
        device.setVolume(device.getVolume() - 10);
    }
    
    public void volumeUp() {
        device.setVolume(device.getVolume() + 10);
    }
    
    public void channelDown() {
        device.setChannel(device.getChannel() - 1);
    }
    
    public void channelUp() {
        device.setChannel(device.getChannel() + 1);
    }
}

// Refined abstraction
class AdvancedRemoteControl extends RemoteControl {
    public AdvancedRemoteControl(Device device) {
        super(device);
    }
    
    public void mute() {
        device.setVolume(0);
        System.out.println("Device muted");
    }
}

// Usage
public class DeviceBridgeExample {
    public static void main(String[] args) {
        // TV with basic remote
        Device tv = new TV();
        RemoteControl tvRemote = new RemoteControl(tv);
        tvRemote.togglePower();
        tvRemote.channelUp();
        tvRemote.volumeUp();
        
        System.out.println();
        
        // Radio with advanced remote
        Device radio = new Radio();
        AdvancedRemoteControl radioRemote = new AdvancedRemoteControl(radio);
        radioRemote.togglePower();
        radioRemote.mute();
    }
}
```

---

## 3. Composite Pattern

### 3.1 Intent

Compose objects into tree structures to represent part-whole hierarchies. Lets clients treat individual objects and compositions uniformly.

### 3.2 When to Use

- Represent part-whole hierarchies
- Want clients to ignore difference between individual objects and compositions
- Need to treat objects and compositions uniformly
- Have tree structure of objects

### 3.3 Scenarios

- **File System**: Files and directories
- **Organization Structure**: Employees and departments
- **UI Components**: Buttons, panels, windows
- **Menu System**: Menu items and submenus
- **Expression Trees**: Operations and operands

### 3.4 Implementation

```java
import java.util.ArrayList;
import java.util.List;

// Component interface
interface FileSystemComponent {
    void display(String indent);
    long getSize();
}

// Leaf
class File implements FileSystemComponent {
    private String name;
    private long size;
    
    public File(String name, long size) {
        this.name = name;
        this.size = size;
    }
    
    @Override
    public void display(String indent) {
        System.out.println(indent + "File: " + name + " (" + size + " bytes)");
    }
    
    @Override
    public long getSize() {
        return size;
    }
}

// Composite
class Directory implements FileSystemComponent {
    private String name;
    private List<FileSystemComponent> children;
    
    public Directory(String name) {
        this.name = name;
        this.children = new ArrayList<>();
    }
    
    public void add(FileSystemComponent component) {
        children.add(component);
    }
    
    public void remove(FileSystemComponent component) {
        children.remove(component);
    }
    
    @Override
    public void display(String indent) {
        System.out.println(indent + "Directory: " + name);
        for (FileSystemComponent component : children) {
            component.display(indent + "  ");
        }
    }
    
    @Override
    public long getSize() {
        long totalSize = 0;
        for (FileSystemComponent component : children) {
            totalSize += component.getSize();
        }
        return totalSize;
    }
}
```

### 3.5 Usage Example

```java
public class CompositeExample {
    public static void main(String[] args) {
        // Create files
        File file1 = new File("document.txt", 1024);
        File file2 = new File("image.jpg", 2048);
        File file3 = new File("video.mp4", 4096);
        
        // Create directories
        Directory root = new Directory("Root");
        Directory documents = new Directory("Documents");
        Directory media = new Directory("Media");
        
        // Build tree structure
        documents.add(file1);
        media.add(file2);
        media.add(file3);
        
        root.add(documents);
        root.add(media);
        
        // Display structure
        root.display("");
        
        System.out.println("\nTotal size: " + root.getSize() + " bytes");
    }
}
```

### 3.6 Real-World Example: Organization Structure

```java
// Component
interface Employee {
    void showDetails();
    double getSalary();
    void add(Employee employee);
    void remove(Employee employee);
}

// Leaf
class IndividualEmployee implements Employee {
    private String name;
    private String position;
    private double salary;
    
    public IndividualEmployee(String name, String position, double salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    
    @Override
    public void showDetails() {
        System.out.println("Employee: " + name + ", Position: " + position + ", Salary: $" + salary);
    }
    
    @Override
    public double getSalary() {
        return salary;
    }
    
    @Override
    public void add(Employee employee) {
        // Leaf cannot have children
        throw new UnsupportedOperationException("Individual employee cannot have subordinates");
    }
    
    @Override
    public void remove(Employee employee) {
        throw new UnsupportedOperationException("Individual employee cannot have subordinates");
    }
}

// Composite
class Department implements Employee {
    private String name;
    private List<Employee> employees;
    
    public Department(String name) {
        this.name = name;
        this.employees = new ArrayList<>();
    }
    
    @Override
    public void showDetails() {
        System.out.println("Department: " + name);
        for (Employee employee : employees) {
            employee.showDetails();
        }
    }
    
    @Override
    public double getSalary() {
        double totalSalary = 0;
        for (Employee employee : employees) {
            totalSalary += employee.getSalary();
        }
        return totalSalary;
    }
    
    @Override
    public void add(Employee employee) {
        employees.add(employee);
    }
    
    @Override
    public void remove(Employee employee) {
        employees.remove(employee);
    }
}

// Usage
public class OrganizationCompositeExample {
    public static void main(String[] args) {
        // Create employees
        Employee emp1 = new IndividualEmployee("John", "Developer", 80000);
        Employee emp2 = new IndividualEmployee("Jane", "Developer", 85000);
        Employee emp3 = new IndividualEmployee("Bob", "Manager", 100000);
        
        // Create departments
        Department engineering = new Department("Engineering");
        engineering.add(emp1);
        engineering.add(emp2);
        
        Department management = new Department("Management");
        management.add(emp3);
        management.add(engineering);
        
        // Display organization
        management.showDetails();
        System.out.println("\nTotal Department Salary: $" + management.getSalary());
    }
}
```

### 3.7 UI Components Example

```java
// Component
interface UIComponent {
    void render();
    void add(UIComponent component);
    void remove(UIComponent component);
}

// Leaf
class Button implements UIComponent {
    private String label;
    
    public Button(String label) {
        this.label = label;
    }
    
    @Override
    public void render() {
        System.out.println("  Rendering Button: " + label);
    }
    
    @Override
    public void add(UIComponent component) {
        throw new UnsupportedOperationException();
    }
    
    @Override
    public void remove(UIComponent component) {
        throw new UnsupportedOperationException();
    }
}

// Composite
class Panel implements UIComponent {
    private String name;
    private List<UIComponent> components;
    
    public Panel(String name) {
        this.name = name;
        this.components = new ArrayList<>();
    }
    
    @Override
    public void render() {
        System.out.println("Rendering Panel: " + name);
        for (UIComponent component : components) {
            component.render();
        }
    }
    
    @Override
    public void add(UIComponent component) {
        components.add(component);
    }
    
    @Override
    public void remove(UIComponent component) {
        components.remove(component);
    }
}

// Usage
public class UICompositeExample {
    public static void main(String[] args) {
        // Create buttons
        UIComponent button1 = new Button("Submit");
        UIComponent button2 = new Button("Cancel");
        UIComponent button3 = new Button("Reset");
        
        // Create panels
        UIComponent formPanel = new Panel("Form Panel");
        formPanel.add(button1);
        formPanel.add(button2);
        
        UIComponent mainPanel = new Panel("Main Panel");
        mainPanel.add(formPanel);
        mainPanel.add(button3);
        
        // Render entire UI tree
        mainPanel.render();
    }
}
```

---

## Summary: Part 4

### Patterns Covered

1. **Proxy**: Control access to objects
2. **Bridge**: Decouple abstraction from implementation
3. **Composite**: Treat objects and compositions uniformly

### Key Takeaways

- **Proxy**: Use for access control, lazy loading, caching
- **Bridge**: Use when abstraction and implementation vary independently
- **Composite**: Use for tree structures and part-whole hierarchies

### Next Steps

**Part 5** will cover:
- Observer Pattern
- Strategy Pattern
- Command Pattern

---

**Master these structural patterns for flexible system design!**

