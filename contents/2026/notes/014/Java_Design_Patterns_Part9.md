# Java Design Patterns: Complete Guide with Examples

## Part 9: Additional Patterns and Modern Patterns

---

## Table of Contents

1. [Dependency Injection Pattern](#1-dependency-injection-pattern)
2. [Repository Pattern](#2-repository-pattern)
3. [Service Locator Pattern](#3-service-locator-pattern)
4. [Data Access Object (DAO) Pattern](#4-data-access-object-dao-pattern)
5. [Model-View-Controller (MVC) Pattern](#5-model-view-controller-mvc-pattern)
6. [Model-View-ViewModel (MVVM) Pattern](#6-model-view-viewmodel-mvvm-pattern)

---

## 1. Dependency Injection Pattern

### 1.1 Intent

A class receives its dependencies from external sources rather than creating them itself.

### 1.2 When to Use

- Want to decouple classes from their dependencies
- Need to test classes in isolation
- Want to change implementations easily
- Need to manage object lifecycle externally
- Want to follow SOLID principles (Dependency Inversion)

### 1.3 Scenarios

- **Service Layer**: Inject repositories, services
- **Testing**: Inject mocks for unit testing
- **Configuration**: Inject configuration objects
- **Framework Integration**: Spring, Guice dependency injection

### 1.4 Implementation: Constructor Injection

```java
// Dependency interface
interface MessageService {
    void sendMessage(String message);
}

// Concrete dependency
class EmailService implements MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending email: " + message);
    }
}

class SMSService implements MessageService {
    @Override
    public void sendMessage(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

// Dependent class (inject dependency via constructor)
class UserService {
    private MessageService messageService;
    
    // Constructor injection
    public UserService(MessageService messageService) {
        this.messageService = messageService;
    }
    
    public void registerUser(String username) {
        System.out.println("Registering user: " + username);
        messageService.sendMessage("Welcome " + username + "!");
    }
}
```

### 1.5 Usage Example

```java
public class DependencyInjectionExample {
    public static void main(String[] args) {
        // Inject EmailService
        MessageService emailService = new EmailService();
        UserService userService1 = new UserService(emailService);
        userService1.registerUser("John");
        
        System.out.println();
        
        // Inject SMSService
        MessageService smsService = new SMSService();
        UserService userService2 = new UserService(smsService);
        userService2.registerUser("Jane");
    }
}
```

### 1.6 Real-World Example: Service Layer

```java
// Repository interface
interface UserRepository {
    void save(User user);
    User findById(String id);
}

// Concrete repository
class DatabaseUserRepository implements UserRepository {
    @Override
    public void save(User user) {
        System.out.println("Saving user to database: " + user.getName());
    }
    
    @Override
    public User findById(String id) {
        System.out.println("Finding user by id: " + id);
        return new User(id, "User " + id);
    }
}

// Service with dependency injection
class UserService {
    private UserRepository userRepository;
    private MessageService messageService;
    
    // Constructor injection
    public UserService(UserRepository userRepository, MessageService messageService) {
        this.userRepository = userRepository;
        this.messageService = messageService;
    }
    
    public void createUser(String id, String name) {
        User user = new User(id, name);
        userRepository.save(user);
        messageService.sendMessage("User created: " + name);
    }
    
    public User getUser(String id) {
        return userRepository.findById(id);
    }
}

class User {
    private String id;
    private String name;
    
    public User(String id, String name) {
        this.id = id;
        this.name = name;
    }
    
    public String getId() { return id; }
    public String getName() { return name; }
}

// Usage
public class ServiceLayerExample {
    public static void main(String[] args) {
        // Inject dependencies
        UserRepository repository = new DatabaseUserRepository();
        MessageService messageService = new EmailService();
        
        UserService userService = new UserService(repository, messageService);
        userService.createUser("1", "John Doe");
    }
}
```

### 1.7 Setter Injection

```java
class OrderService {
    private PaymentService paymentService;
    private ShippingService shippingService;
    
    // Setter injection
    public void setPaymentService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
    
    public void setShippingService(ShippingService shippingService) {
        this.shippingService = shippingService;
    }
    
    public void processOrder(Order order) {
        paymentService.processPayment(order.getAmount());
        shippingService.ship(order);
    }
}
```

---

## 2. Repository Pattern

### 2.1 Intent

Mediates between the domain and data mapping layers, acting like an in-memory domain object collection.

### 2.2 When to Use

- Need to abstract data access logic
- Want to make code testable
- Need to switch data sources easily
- Want to centralize data access logic
- Need to implement caching or query optimization

### 2.3 Scenarios

- **Data Access**: Abstract database operations
- **Testing**: Mock repositories for unit tests
- **Multiple Data Sources**: Switch between database, file, API
- **Caching**: Add caching layer in repository

### 2.4 Implementation

```java
import java.util.*;

// Entity
class User {
    private String id;
    private String name;
    private String email;
    
    public User(String id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    
    // Getters and setters
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    @Override
    public String toString() {
        return "User{id='" + id + "', name='" + name + "', email='" + email + "'}";
    }
}

// Repository interface
interface UserRepository {
    User save(User user);
    Optional<User> findById(String id);
    List<User> findAll();
    void deleteById(String id);
    List<User> findByName(String name);
}

// Concrete repository implementation
class InMemoryUserRepository implements UserRepository {
    private Map<String, User> users = new HashMap<>();
    
    @Override
    public User save(User user) {
        users.put(user.getId(), user);
        System.out.println("Saved: " + user);
        return user;
    }
    
    @Override
    public Optional<User> findById(String id) {
        User user = users.get(id);
        System.out.println("Found by ID " + id + ": " + user);
        return Optional.ofNullable(user);
    }
    
    @Override
    public List<User> findAll() {
        return new ArrayList<>(users.values());
    }
    
    @Override
    public void deleteById(String id) {
        User removed = users.remove(id);
        System.out.println("Deleted: " + removed);
    }
    
    @Override
    public List<User> findByName(String name) {
        List<User> result = new ArrayList<>();
        for (User user : users.values()) {
            if (user.getName().equals(name)) {
                result.add(user);
            }
        }
        return result;
    }
}
```

### 2.5 Usage Example

```java
public class RepositoryExample {
    public static void main(String[] args) {
        UserRepository repository = new InMemoryUserRepository();
        
        // Save users
        repository.save(new User("1", "John Doe", "john@example.com"));
        repository.save(new User("2", "Jane Smith", "jane@example.com"));
        repository.save(new User("3", "John Doe", "john2@example.com"));
        
        // Find operations
        repository.findById("1").ifPresent(System.out::println);
        System.out.println("All users: " + repository.findAll());
        System.out.println("Users named John: " + repository.findByName("John Doe"));
        
        // Delete
        repository.deleteById("2");
        System.out.println("After delete: " + repository.findAll());
    }
}
```

### 2.6 Real-World Example: Database Repository

```java
import java.sql.*;
import java.util.*;

// Database repository
class DatabaseUserRepository implements UserRepository {
    private Connection connection;
    
    public DatabaseUserRepository(Connection connection) {
        this.connection = connection;
    }
    
    @Override
    public User save(User user) {
        try {
            String sql = "INSERT INTO users (id, name, email) VALUES (?, ?, ?) " +
                        "ON CONFLICT (id) DO UPDATE SET name = ?, email = ?";
            PreparedStatement stmt = connection.prepareStatement(sql);
            stmt.setString(1, user.getId());
            stmt.setString(2, user.getName());
            stmt.setString(3, user.getEmail());
            stmt.setString(4, user.getName());
            stmt.setString(5, user.getEmail());
            stmt.executeUpdate();
            return user;
        } catch (SQLException e) {
            throw new RuntimeException("Error saving user", e);
        }
    }
    
    @Override
    public Optional<User> findById(String id) {
        try {
            String sql = "SELECT * FROM users WHERE id = ?";
            PreparedStatement stmt = connection.prepareStatement(sql);
            stmt.setString(1, id);
            ResultSet rs = stmt.executeQuery();
            
            if (rs.next()) {
                return Optional.of(new User(
                    rs.getString("id"),
                    rs.getString("name"),
                    rs.getString("email")
                ));
            }
            return Optional.empty();
        } catch (SQLException e) {
            throw new RuntimeException("Error finding user", e);
        }
    }
    
    @Override
    public List<User> findAll() {
        List<User> users = new ArrayList<>();
        try {
            String sql = "SELECT * FROM users";
            Statement stmt = connection.createStatement();
            ResultSet rs = stmt.executeQuery(sql);
            
            while (rs.next()) {
                users.add(new User(
                    rs.getString("id"),
                    rs.getString("name"),
                    rs.getString("email")
                ));
            }
        } catch (SQLException e) {
            throw new RuntimeException("Error finding all users", e);
        }
        return users;
    }
    
    @Override
    public void deleteById(String id) {
        try {
            String sql = "DELETE FROM users WHERE id = ?";
            PreparedStatement stmt = connection.prepareStatement(sql);
            stmt.setString(1, id);
            stmt.executeUpdate();
        } catch (SQLException e) {
            throw new RuntimeException("Error deleting user", e);
        }
    }
    
    @Override
    public List<User> findByName(String name) {
        List<User> users = new ArrayList<>();
        try {
            String sql = "SELECT * FROM users WHERE name = ?";
            PreparedStatement stmt = connection.prepareStatement(sql);
            stmt.setString(1, name);
            ResultSet rs = stmt.executeQuery();
            
            while (rs.next()) {
                users.add(new User(
                    rs.getString("id"),
                    rs.getString("name"),
                    rs.getString("email")
                ));
            }
        } catch (SQLException e) {
            throw new RuntimeException("Error finding users by name", e);
        }
        return users;
    }
}
```

---

## 3. Service Locator Pattern

### 3.1 Intent

Encapsulate the processes involved in obtaining a service with a strong abstraction layer.

### 3.2 When to Use

- Need centralized service lookup
- Want to decouple service consumers from service implementations
- Need to manage service lifecycle
- Want to provide service caching

### 3.3 Scenarios

- **Service Discovery**: Locate services in distributed systems
- **Plugin Architecture**: Locate and load plugins
- **Legacy Integration**: Integrate with legacy systems
- **Service Registry**: Centralized service registry

### 3.4 Implementation

```java
import java.util.*;

// Service interface
interface Service {
    String getName();
    void execute();
}

// Concrete services
class EmailService implements Service {
    @Override
    public String getName() {
        return "EmailService";
    }
    
    @Override
    public void execute() {
        System.out.println("Executing EmailService");
    }
}

class SMSService implements Service {
    @Override
    public String getName() {
        return "SMSService";
    }
    
    @Override
    public void execute() {
        System.out.println("Executing SMSService");
    }
}

// Service locator
class ServiceLocator {
    private static ServiceLocator instance;
    private Map<String, Service> services;
    
    private ServiceLocator() {
        this.services = new HashMap<>();
    }
    
    public static ServiceLocator getInstance() {
        if (instance == null) {
            instance = new ServiceLocator();
        }
        return instance;
    }
    
    public void registerService(String name, Service service) {
        services.put(name, service);
    }
    
    public Service getService(String serviceName) {
        Service service = services.get(serviceName);
        if (service == null) {
            throw new RuntimeException("Service not found: " + serviceName);
        }
        return service;
    }
}
```

### 3.5 Usage Example

```java
public class ServiceLocatorExample {
    public static void main(String[] args) {
        ServiceLocator locator = ServiceLocator.getInstance();
        
        // Register services
        locator.registerService("EmailService", new EmailService());
        locator.registerService("SMSService", new SMSService());
        
        // Get and use services
        Service emailService = locator.getService("EmailService");
        emailService.execute();
        
        Service smsService = locator.getService("SMSService");
        smsService.execute();
    }
}
```

---

## 4. Data Access Object (DAO) Pattern

### 4.1 Intent

Separate low-level data accessing operations from high-level business logic.

### 4.2 When to Use

- Need to abstract data access logic
- Want to separate business logic from data access
- Need to support multiple data sources
- Want to make code testable

### 4.3 Scenarios

- **Database Operations**: CRUD operations
- **File Operations**: File read/write operations
- **API Operations**: External API data access
- **Caching**: Cache data access

### 4.4 Implementation

```java
import java.util.*;

// Entity
class Product {
    private int id;
    private String name;
    private double price;
    
    public Product(int id, String name, double price) {
        this.id = id;
        this.name = name;
        this.price = price;
    }
    
    // Getters and setters
    public int getId() { return id; }
    public void setId(int id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public double getPrice() { return price; }
    public void setPrice(double price) { this.price = price; }
    
    @Override
    public String toString() {
        return "Product{id=" + id + ", name='" + name + "', price=" + price + "}";
    }
}

// DAO interface
interface ProductDAO {
    List<Product> getAllProducts();
    Product getProduct(int id);
    void addProduct(Product product);
    void updateProduct(Product product);
    void deleteProduct(int id);
}

// Concrete DAO
class InMemoryProductDAO implements ProductDAO {
    private List<Product> products;
    
    public InMemoryProductDAO() {
        this.products = new ArrayList<>();
    }
    
    @Override
    public List<Product> getAllProducts() {
        return new ArrayList<>(products);
    }
    
    @Override
    public Product getProduct(int id) {
        return products.stream()
                .filter(p -> p.getId() == id)
                .findFirst()
                .orElse(null);
    }
    
    @Override
    public void addProduct(Product product) {
        products.add(product);
        System.out.println("Product added: " + product);
    }
    
    @Override
    public void updateProduct(Product product) {
        Product existing = getProduct(product.getId());
        if (existing != null) {
            existing.setName(product.getName());
            existing.setPrice(product.getPrice());
            System.out.println("Product updated: " + existing);
        }
    }
    
    @Override
    public void deleteProduct(int id) {
        products.removeIf(p -> p.getId() == id);
        System.out.println("Product deleted: " + id);
    }
}
```

### 4.5 Usage Example

```java
public class DAOExample {
    public static void main(String[] args) {
        ProductDAO productDAO = new InMemoryProductDAO();
        
        // Add products
        productDAO.addProduct(new Product(1, "Laptop", 999.99));
        productDAO.addProduct(new Product(2, "Mouse", 29.99));
        productDAO.addProduct(new Product(3, "Keyboard", 79.99));
        
        // Get all products
        System.out.println("All products:");
        productDAO.getAllProducts().forEach(System.out::println);
        
        // Get product by ID
        System.out.println("\nProduct with ID 2:");
        System.out.println(productDAO.getProduct(2));
        
        // Update product
        productDAO.updateProduct(new Product(2, "Wireless Mouse", 39.99));
        
        // Delete product
        productDAO.deleteProduct(3);
        
        System.out.println("\nRemaining products:");
        productDAO.getAllProducts().forEach(System.out::println);
    }
}
```

---

## 5. Model-View-Controller (MVC) Pattern

### 5.1 Intent

Separate application into three interconnected components: Model (data), View (presentation), Controller (logic).

### 5.2 When to Use

- Need to separate concerns
- Want to support multiple views
- Need to make views independent of data
- Want to make code maintainable

### 5.3 Scenarios

- **Web Applications**: Spring MVC, JSF
- **Desktop Applications**: Swing, JavaFX
- **Mobile Applications**: Android MVC
- **API Design**: RESTful APIs

### 5.4 Implementation

```java
// Model
class Student {
    private String rollNo;
    private String name;
    
    public Student(String rollNo, String name) {
        this.rollNo = rollNo;
        this.name = name;
    }
    
    public String getRollNo() { return rollNo; }
    public void setRollNo(String rollNo) { this.rollNo = rollNo; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

// View
class StudentView {
    public void printStudentDetails(String studentName, String studentRollNo) {
        System.out.println("Student:");
        System.out.println("Name: " + studentName);
        System.out.println("Roll No: " + studentRollNo);
    }
}

// Controller
class StudentController {
    private Student model;
    private StudentView view;
    
    public StudentController(Student model, StudentView view) {
        this.model = model;
        this.view = view;
    }
    
    public void setStudentName(String name) {
        model.setName(name);
    }
    
    public String getStudentName() {
        return model.getName();
    }
    
    public void setStudentRollNo(String rollNo) {
        model.setRollNo(rollNo);
    }
    
    public String getStudentRollNo() {
        return model.getRollNo();
    }
    
    public void updateView() {
        view.printStudentDetails(model.getName(), model.getRollNo());
    }
}
```

### 5.5 Usage Example

```java
public class MVCExample {
    public static void main(String[] args) {
        // Create model
        Student model = retrieveStudentFromDatabase();
        
        // Create view
        StudentView view = new StudentView();
        
        // Create controller
        StudentController controller = new StudentController(model, view);
        
        controller.updateView();
        
        // Update model
        controller.setStudentName("John Updated");
        controller.updateView();
    }
    
    private static Student retrieveStudentFromDatabase() {
        return new Student("1", "John Doe");
    }
}
```

### 5.6 Real-World Example: Web MVC

```java
// Model
class User {
    private String id;
    private String username;
    private String email;
    
    public User(String id, String username, String email) {
        this.id = id;
        this.username = username;
        this.email = email;
    }
    
    // Getters and setters
    public String getId() { return id; }
    public String getUsername() { return username; }
    public String getEmail() { return email; }
}

// View
interface UserView {
    void displayUser(User user);
    void displayError(String message);
}

class ConsoleUserView implements UserView {
    @Override
    public void displayUser(User user) {
        System.out.println("User Details:");
        System.out.println("ID: " + user.getId());
        System.out.println("Username: " + user.getUsername());
        System.out.println("Email: " + user.getEmail());
    }
    
    @Override
    public void displayError(String message) {
        System.err.println("Error: " + message);
    }
}

// Controller
class UserController {
    private User model;
    private UserView view;
    
    public UserController(User model, UserView view) {
        this.model = model;
        this.view = view;
    }
    
    public void setUserUsername(String username) {
        model = new User(model.getId(), username, model.getEmail());
    }
    
    public void setUserEmail(String email) {
        model = new User(model.getId(), model.getUsername(), email);
    }
    
    public String getUserId() {
        return model.getId();
    }
    
    public String getUsername() {
        return model.getUsername();
    }
    
    public String getUserEmail() {
        return model.getEmail();
    }
    
    public void updateView() {
        view.displayUser(model);
    }
}

// Usage
public class WebMVCExample {
    public static void main(String[] args) {
        User model = new User("1", "johndoe", "john@example.com");
        UserView view = new ConsoleUserView();
        UserController controller = new UserController(model, view);
        
        controller.updateView();
        
        controller.setUserEmail("john.updated@example.com");
        controller.updateView();
    }
}
```

---

## 6. Model-View-ViewModel (MVVM) Pattern

### 6.1 Intent

Separate the development of graphical user interface from business logic through a view model.

### 6.2 When to Use

- Building UI applications
- Want to separate view logic from business logic
- Need data binding
- Want to make views testable

### 6.3 Scenarios

- **JavaFX Applications**: Desktop applications
- **Android Applications**: MVVM architecture
- **Web Applications**: Frontend frameworks
- **Data Binding**: Two-way data binding

### 6.4 Implementation

```java
import java.beans.PropertyChangeListener;
import java.beans.PropertyChangeSupport;

// Model
class UserModel {
    private String name;
    private String email;
    
    public UserModel(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
}

// ViewModel
class UserViewModel {
    private UserModel model;
    private PropertyChangeSupport pcs = new PropertyChangeSupport(this);
    
    public UserViewModel(UserModel model) {
        this.model = model;
    }
    
    public void addPropertyChangeListener(PropertyChangeListener listener) {
        pcs.addPropertyChangeListener(listener);
    }
    
    public String getName() {
        return model.getName();
    }
    
    public void setName(String name) {
        String oldName = model.getName();
        model.setName(name);
        pcs.firePropertyChange("name", oldName, name);
    }
    
    public String getEmail() {
        return model.getEmail();
    }
    
    public void setEmail(String email) {
        String oldEmail = model.getEmail();
        model.setEmail(email);
        pcs.firePropertyChange("email", oldEmail, email);
    }
    
    public void updateUser(String name, String email) {
        setName(name);
        setEmail(email);
    }
}

// View
class UserView {
    private UserViewModel viewModel;
    
    public UserView(UserViewModel viewModel) {
        this.viewModel = viewModel;
        setupBindings();
    }
    
    private void setupBindings() {
        viewModel.addPropertyChangeListener(evt -> {
            System.out.println("Property changed: " + evt.getPropertyName() + 
                             " = " + evt.getNewValue());
            display();
        });
    }
    
    public void display() {
        System.out.println("View Display:");
        System.out.println("Name: " + viewModel.getName());
        System.out.println("Email: " + viewModel.getEmail());
    }
    
    public void updateName(String name) {
        viewModel.setName(name);
    }
    
    public void updateEmail(String email) {
        viewModel.setEmail(email);
    }
}

// Usage
public class MVVMExample {
    public static void main(String[] args) {
        UserModel model = new UserModel("John Doe", "john@example.com");
        UserViewModel viewModel = new UserViewModel(model);
        UserView view = new UserView(viewModel);
        
        view.display();
        
        System.out.println("\nUpdating name:");
        view.updateName("Jane Smith");
        
        System.out.println("\nUpdating email:");
        view.updateEmail("jane@example.com");
    }
}
```

---

## Summary: Part 9

### Patterns Covered

1. **Dependency Injection**: Inject dependencies externally
2. **Repository**: Abstract data access
3. **Service Locator**: Centralized service lookup
4. **DAO**: Separate data access from business logic
5. **MVC**: Separate Model, View, Controller
6. **MVVM**: Separate Model, View, ViewModel

### Key Takeaways

- **Dependency Injection**: Use for loose coupling and testability
- **Repository/DAO**: Use to abstract data access
- **MVC/MVVM**: Use for UI architecture

### Next Steps

**Part 10** will cover:
- Pattern Selection Guide
- Best Practices
- Anti-Patterns
- Complete Pattern Reference

---

**Master these additional patterns for enterprise application design!**

