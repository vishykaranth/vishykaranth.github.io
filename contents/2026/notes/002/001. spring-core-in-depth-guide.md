# Spring Core In-Depth Interview Guide: Dependency Injection, IoC Container & Bean Lifecycle

## Table of Contents
1. [Spring Framework Overview](#spring-framework-overview)
2. [Inversion of Control (IoC)](#inversion-of-control-ioc)
3. [Dependency Injection (DI)](#dependency-injection-di)
4. [Spring IoC Container](#spring-ioc-container)
5. [Bean Definition & Configuration](#bean-definition--configuration)
6. [Bean Lifecycle](#bean-lifecycle)
7. [Bean Scopes](#bean-scopes)
8. [Advanced Topics](#advanced-topics)
9. [Best Practices](#best-practices)
10. [Interview Questions & Answers](#interview-questions--answers)

---

## Spring Framework Overview

### What is Spring Framework?

**Spring Framework** is a comprehensive Java application framework that provides:
- **Dependency Injection (DI)**: Loose coupling through IoC
- **Aspect-Oriented Programming (AOP)**: Cross-cutting concerns
- **Transaction Management**: Declarative transactions
- **MVC Framework**: Web application development
- **Integration**: Messaging, scheduling, caching

### Core Principles

1. **Inversion of Control (IoC)**: Framework manages object creation and dependencies
2. **Dependency Injection (DI)**: Dependencies are injected, not created by objects
3. **Aspect-Oriented Programming (AOP)**: Separation of cross-cutting concerns
4. **Convention over Configuration**: Sensible defaults reduce configuration

### Spring Modules

- **Spring Core**: IoC container, DI
- **Spring AOP**: Aspect-oriented programming
- **Spring Context**: Application context, internationalization
- **Spring Beans**: Bean factory, bean definitions
- **Spring Expression Language (SpEL)**: Expression evaluation

---

## Inversion of Control (IoC)

### What is IoC?

**Inversion of Control (IoC)** is a design principle where:
- **Traditional Approach**: Objects create their own dependencies
- **IoC Approach**: Framework/container creates and injects dependencies
- **Control is Inverted**: From objects to framework

### IoC Benefits

1. **Loose Coupling**: Objects depend on abstractions, not implementations
2. **Testability**: Easy to mock dependencies
3. **Flexibility**: Change implementations without modifying code
4. **Maintainability**: Centralized dependency management

### Example: Without IoC

```java
// Traditional approach - Tight coupling
public class OrderService {
    private PaymentProcessor paymentProcessor;
    private EmailService emailService;
    
    public OrderService() {
        // Object creates its own dependencies - BAD!
        this.paymentProcessor = new CreditCardProcessor();
        this.emailService = new SmtpEmailService();
    }
    
    public void processOrder(Order order) {
        paymentProcessor.process(order.getAmount());
        emailService.sendConfirmation(order);
    }
}
```

**Problems:**
- Tight coupling to concrete implementations
- Hard to test (can't mock dependencies)
- Hard to change implementations
- Violates Dependency Inversion Principle

### Example: With IoC

```java
// IoC approach - Loose coupling
public class OrderService {
    private PaymentProcessor paymentProcessor;
    private EmailService emailService;
    
    // Dependencies injected via constructor
    public OrderService(PaymentProcessor paymentProcessor, EmailService emailService) {
        this.paymentProcessor = paymentProcessor;
        this.emailService = emailService;
    }
    
    public void processOrder(Order order) {
        paymentProcessor.process(order.getAmount());
        emailService.sendConfirmation(order);
    }
}

// Spring manages dependencies
@Configuration
public class AppConfig {
    @Bean
    public PaymentProcessor paymentProcessor() {
        return new CreditCardProcessor();
    }
    
    @Bean
    public EmailService emailService() {
        return new SmtpEmailService();
    }
    
    @Bean
    public OrderService orderService(PaymentProcessor paymentProcessor, EmailService emailService) {
        return new OrderService(paymentProcessor, emailService);
    }
}
```

**Benefits:**
- Loose coupling to interfaces
- Easy to test (inject mocks)
- Easy to change implementations
- Follows Dependency Inversion Principle

---

## Dependency Injection (DI)

### What is Dependency Injection?

**Dependency Injection (DI)** is a design pattern where:
- Dependencies are **provided** to objects (injected)
- Objects don't create their own dependencies
- Framework/container manages dependency resolution

### DI Types

#### 1. **Constructor Injection** (Recommended)

**Benefits:**
- Immutable dependencies (final fields)
- Required dependencies (fail-fast if missing)
- Easier testing
- No circular dependency issues

```java
@Service
public class UserService {
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // Constructor injection
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    public void createUser(User user) {
        userRepository.save(user);
        emailService.sendWelcomeEmail(user);
    }
}
```

**Configuration:**

```java
@Configuration
public class AppConfig {
    @Bean
    public UserRepository userRepository() {
        return new JpaUserRepository();
    }
    
    @Bean
    public EmailService emailService() {
        return new SmtpEmailService();
    }
    
    @Bean
    public UserService userService(UserRepository userRepository, EmailService emailService) {
        return new UserService(userRepository, emailService);
    }
}
```

#### 2. **Setter Injection**

**Use When:**
- Optional dependencies
- Need to change dependencies after construction
- Legacy code compatibility

```java
@Service
public class OrderService {
    private PaymentProcessor paymentProcessor;
    private EmailService emailService;
    
    // Setter injection
    @Autowired
    public void setPaymentProcessor(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
    
    @Autowired
    public void setEmailService(EmailService emailService) {
        this.emailService = emailService;
    }
    
    public void processOrder(Order order) {
        paymentProcessor.process(order.getAmount());
        emailService.sendConfirmation(order);
    }
}
```

**Configuration:**

```java
@Configuration
public class AppConfig {
    @Bean
    public OrderService orderService() {
        OrderService service = new OrderService();
        // Spring will call setters automatically
        return service;
    }
}
```

#### 3. **Field Injection** (Not Recommended)

**Problems:**
- Hard to test (requires reflection or Spring context)
- Can't make fields final
- Hides dependencies
- Circular dependency issues

```java
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;  // Not recommended!
    
    @Autowired
    private EmailService emailService;  // Not recommended!
}
```

**Why Not Recommended:**
- Can't make fields `final` (immutability)
- Hard to unit test without Spring context
- Dependencies are hidden
- Violates encapsulation

#### 4. **Method Injection** (Rare)

```java
@Service
public class OrderService {
    private PaymentProcessor paymentProcessor;
    
    @Autowired
    public void configure(PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
}
```

### Dependency Resolution

#### By Type (Default)

```java
@Configuration
public class AppConfig {
    @Bean
    public PaymentProcessor creditCardProcessor() {
        return new CreditCardProcessor();
    }
    
    @Bean
    public OrderService orderService(PaymentProcessor paymentProcessor) {
        // Spring finds PaymentProcessor by type
        return new OrderService(paymentProcessor);
    }
}
```

#### By Name

```java
@Configuration
public class AppConfig {
    @Bean("creditCardProcessor")
    public PaymentProcessor creditCardProcessor() {
        return new CreditCardProcessor();
    }
    
    @Bean("paypalProcessor")
    public PaymentProcessor paypalProcessor() {
        return new PaypalProcessor();
    }
    
    @Bean
    public OrderService orderService(@Qualifier("creditCardProcessor") PaymentProcessor processor) {
        return new OrderService(processor);
    }
}
```

#### Using @Qualifier

```java
@Service
public class OrderService {
    private final PaymentProcessor paymentProcessor;
    
    public OrderService(@Qualifier("creditCardProcessor") PaymentProcessor paymentProcessor) {
        this.paymentProcessor = paymentProcessor;
    }
}
```

#### Using @Primary

```java
@Configuration
public class AppConfig {
    @Bean
    @Primary  // Default when multiple beans of same type
    public PaymentProcessor creditCardProcessor() {
        return new CreditCardProcessor();
    }
    
    @Bean
    public PaymentProcessor paypalProcessor() {
        return new PaypalProcessor();
    }
}
```

### Circular Dependencies

**Problem:** Two beans depend on each other

```java
@Service
public class ServiceA {
    private final ServiceB serviceB;
    
    public ServiceA(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}

@Service
public class ServiceB {
    private final ServiceA serviceA;  // Circular dependency!
    
    public ServiceB(ServiceA serviceA) {
        this.serviceA = serviceA;
    }
}
```

**Solutions:**

1. **Refactor**: Remove circular dependency (best)
2. **Setter Injection**: Use setter for one dependency
3. **@Lazy**: Lazy initialization

```java
@Service
public class ServiceA {
    private ServiceB serviceB;
    
    @Autowired
    @Lazy  // Lazy initialization breaks cycle
    public void setServiceB(ServiceB serviceB) {
        this.serviceB = serviceB;
    }
}
```

---

## Spring IoC Container

### Container Types

#### 1. **BeanFactory** (Basic Container)

**Features:**
- Lazy bean initialization
- Lightweight
- Basic DI support
- Manual bean registration

```java
// Programmatic configuration
DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
reader.loadBeanDefinitions(new ClassPathResource("beans.xml"));

// Get bean
UserService userService = factory.getBean(UserService.class);
```

#### 2. **ApplicationContext** (Advanced Container)

**Features:**
- Eager bean initialization
- Event publishing
- Internationalization
- Application-layer specific contexts
- Automatic BeanFactoryPostProcessor and BeanPostProcessor registration

**Types of ApplicationContext:**

1. **AnnotationConfigApplicationContext**
```java
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
UserService userService = context.getBean(UserService.class);
```

2. **ClassPathXmlApplicationContext**
```java
ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
UserService userService = context.getBean(UserService.class);
```

3. **FileSystemXmlApplicationContext**
```java
ApplicationContext context = new FileSystemXmlApplicationContext("C:/config/applicationContext.xml");
```

4. **WebApplicationContext** (Spring MVC)
```java
// Automatically created by Spring MVC
// Access via @Autowired in controllers
```

### Container Initialization Process

```
1. Load Configuration
   ↓
2. Parse Bean Definitions
   ↓
3. Register Bean Definitions
   ↓
4. Instantiate Beans (if eager)
   ↓
5. Inject Dependencies
   ↓
6. Initialize Beans (PostProcessors)
   ↓
7. Container Ready
```

### Hands-On: Creating Application Context

```java
// Java-based configuration
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    // Bean definitions
}

// Create context
ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);

// Get beans
UserService userService = context.getBean(UserService.class);
UserService userService2 = context.getBean("userService", UserService.class);

// Get all beans of type
Map<String, PaymentProcessor> processors = context.getBeansOfType(PaymentProcessor.class);

// Check if bean exists
boolean exists = context.containsBean("userService");

// Get bean names
String[] beanNames = context.getBeanNamesForType(UserService.class);
```

---

## Bean Definition & Configuration

### Configuration Methods

#### 1. **XML Configuration** (Legacy)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="userRepository" class="com.example.JpaUserRepository"/>
    
    <bean id="emailService" class="com.example.SmtpEmailService"/>
    
    <bean id="userService" class="com.example.UserService">
        <constructor-arg ref="userRepository"/>
        <constructor-arg ref="emailService"/>
    </bean>
</beans>
```

#### 2. **Java-based Configuration** (Recommended)

```java
@Configuration
public class AppConfig {
    
    @Bean
    public UserRepository userRepository() {
        return new JpaUserRepository();
    }
    
    @Bean
    public EmailService emailService() {
        return new SmtpEmailService();
    }
    
    @Bean
    public UserService userService(UserRepository userRepository, EmailService emailService) {
        return new UserService(userRepository, emailService);
    }
}
```

#### 3. **Annotation-based Configuration** (Most Common)

```java
// Enable component scanning
@Configuration
@ComponentScan(basePackages = "com.example")
public class AppConfig {
    // Additional @Bean methods if needed
}

// Use annotations in classes
@Repository
public class JpaUserRepository implements UserRepository {
    // Implementation
}

@Service
public class UserService {
    private final UserRepository userRepository;
    
    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

### Stereotype Annotations

#### @Component
```java
@Component
public class MyComponent {
    // Generic Spring component
}
```

#### @Service
```java
@Service
public class UserService {
    // Business logic service
}
```

#### @Repository
```java
@Repository
public class UserRepository {
    // Data access layer
    // Exception translation enabled
}
```

#### @Controller
```java
@Controller
public class UserController {
    // Web controller (Spring MVC)
}
```

#### Custom Stereotype
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Component
public @interface MyCustomComponent {
    // Custom annotation
}

@MyCustomComponent
public class MyCustomService {
    // Will be detected as component
}
```

### Bean Naming

**Default Naming:**
- Class name with first letter lowercase
- `UserService` → `userService`
- `MyService` → `myService`

**Custom Naming:**
```java
@Service("customUserService")
public class UserService {
    // Bean name: customUserService
}

@Component("myComponent")
public class MyComponent {
    // Bean name: myComponent
}
```

### Conditional Bean Creation

#### @ConditionalOnProperty
```java
@Configuration
public class AppConfig {
    
    @Bean
    @ConditionalOnProperty(name = "feature.email.enabled", havingValue = "true")
    public EmailService emailService() {
        return new SmtpEmailService();
    }
    
    @Bean
    @ConditionalOnProperty(name = "feature.email.enabled", havingValue = "false", matchIfMissing = true)
    public EmailService noOpEmailService() {
        return new NoOpEmailService();
    }
}
```

#### @ConditionalOnClass
```java
@Bean
@ConditionalOnClass(name = "com.example.SpecialClass")
public SpecialService specialService() {
    return new SpecialService();
}
```

#### @Profile
```java
@Configuration
@Profile("production")
public class ProductionConfig {
    @Bean
    public DataSource productionDataSource() {
        return new ProductionDataSource();
    }
}

@Configuration
@Profile("development")
public class DevelopmentConfig {
    @Bean
    public DataSource developmentDataSource() {
        return new H2DataSource();
    }
}
```

---

## Bean Lifecycle

### Bean Lifecycle Phases

```
1. Instantiation
   ↓
2. Populate Properties (Dependency Injection)
   ↓
3. BeanNameAware.setBeanName()
   ↓
4. BeanFactoryAware.setBeanFactory()
   ↓
5. ApplicationContextAware.setApplicationContext()
   ↓
6. BeanPostProcessor.postProcessBeforeInitialization()
   ↓
7. @PostConstruct / InitializingBean.afterPropertiesSet()
   ↓
8. BeanPostProcessor.postProcessAfterInitialization()
   ↓
9. Bean Ready (In Use)
   ↓
10. @PreDestroy / DisposableBean.destroy()
   ↓
11. Bean Destroyed
```

### Lifecycle Callbacks

#### 1. **@PostConstruct** (Recommended)

```java
@Service
public class UserService {
    private UserRepository userRepository;
    
    @PostConstruct
    public void init() {
        System.out.println("UserService initialized");
        // Custom initialization logic
        // Database connections, cache warm-up, etc.
    }
}
```

#### 2. **InitializingBean Interface**

```java
@Service
public class UserService implements InitializingBean {
    private UserRepository userRepository;
    
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("UserService initialized via InitializingBean");
        // Initialization logic
    }
}
```

#### 3. **Custom Init Method**

```java
@Configuration
public class AppConfig {
    @Bean(initMethod = "customInit")
    public UserService userService() {
        return new UserService();
    }
}

public class UserService {
    public void customInit() {
        System.out.println("Custom init method called");
    }
}
```

#### 4. **@PreDestroy** (Recommended)

```java
@Service
public class UserService {
    private UserRepository userRepository;
    
    @PreDestroy
    public void cleanup() {
        System.out.println("UserService being destroyed");
        // Cleanup logic
        // Close connections, release resources, etc.
    }
}
```

#### 5. **DisposableBean Interface**

```java
@Service
public class UserService implements DisposableBean {
    private UserRepository userRepository;
    
    @Override
    public void destroy() throws Exception {
        System.out.println("UserService destroyed via DisposableBean");
        // Cleanup logic
    }
}
```

#### 6. **Custom Destroy Method**

```java
@Configuration
public class AppConfig {
    @Bean(destroyMethod = "customDestroy")
    public UserService userService() {
        return new UserService();
    }
}

public class UserService {
    public void customDestroy() {
        System.out.println("Custom destroy method called");
    }
}
```

### Complete Lifecycle Example

```java
@Service
public class LifecycleDemoService 
        implements BeanNameAware, BeanFactoryAware, ApplicationContextAware,
                   InitializingBean, DisposableBean {
    
    private String beanName;
    private BeanFactory beanFactory;
    private ApplicationContext applicationContext;
    
    // 1. Constructor
    public LifecycleDemoService() {
        System.out.println("1. Constructor called");
    }
    
    // 2. BeanNameAware
    @Override
    public void setBeanName(String name) {
        this.beanName = name;
        System.out.println("2. setBeanName called: " + name);
    }
    
    // 3. BeanFactoryAware
    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        this.beanFactory = beanFactory;
        System.out.println("3. setBeanFactory called");
    }
    
    // 4. ApplicationContextAware
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
        System.out.println("4. setApplicationContext called");
    }
    
    // 5. @PostConstruct
    @PostConstruct
    public void postConstruct() {
        System.out.println("5. @PostConstruct called");
    }
    
    // 6. InitializingBean
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("6. afterPropertiesSet called");
    }
    
    // 7. Custom init method
    public void customInit() {
        System.out.println("7. Custom init method called");
    }
    
    // 8. @PreDestroy
    @PreDestroy
    public void preDestroy() {
        System.out.println("8. @PreDestroy called");
    }
    
    // 9. DisposableBean
    @Override
    public void destroy() throws Exception {
        System.out.println("9. destroy() called");
    }
    
    // 10. Custom destroy method
    public void customDestroy() {
        System.out.println("10. Custom destroy method called");
    }
}
```

### BeanPostProcessor

**Custom BeanPostProcessor:**

```java
@Component
public class CustomBeanPostProcessor implements BeanPostProcessor {
    
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("Before initialization: " + beanName);
        // Modify bean before initialization
        return bean;
    }
    
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("After initialization: " + beanName);
        // Modify bean after initialization
        // Can return proxy, wrapper, etc.
        return bean;
    }
}
```

**Use Cases:**
- Logging
- Performance monitoring
- Proxy creation (AOP)
- Validation
- Custom annotations processing

---

## Bean Scopes

### Available Scopes

#### 1. **Singleton** (Default)

**Characteristics:**
- One instance per container
- Shared across all requests
- Created at container startup (if eager)
- Thread-safe implementation required

```java
@Service
@Scope("singleton")  // Default, can be omitted
public class UserService {
    // Single instance shared across application
}
```

#### 2. **Prototype**

**Characteristics:**
- New instance for each request
- Created when requested
- Not managed by container after creation
- No destroy callbacks

```java
@Service
@Scope("prototype")
public class UserService {
    // New instance each time
}
```

#### 3. **Request** (Web)

**Characteristics:**
- One instance per HTTP request
- Only in web context
- Destroyed at end of request

```java
@Component
@Scope("request")
public class RequestScopedBean {
    // One per HTTP request
}
```

#### 4. **Session** (Web)

**Characteristics:**
- One instance per HTTP session
- Only in web context
- Destroyed when session ends

```java
@Component
@Scope("session")
public class SessionScopedBean {
    // One per HTTP session
}
```

#### 5. **Application** (Web)

**Characteristics:**
- One instance per ServletContext
- Similar to singleton but web-aware

```java
@Component
@Scope("application")
public class ApplicationScopedBean {
    // One per ServletContext
}
```

#### 6. **WebSocket** (Web)

**Characteristics:**
- One instance per WebSocket session
- Only in web context

```java
@Component
@Scope("websocket")
public class WebSocketScopedBean {
    // One per WebSocket session
}
```

### Scope Example

```java
@Configuration
public class ScopeConfig {
    
    @Bean
    @Scope("singleton")
    public SingletonBean singletonBean() {
        return new SingletonBean();
    }
    
    @Bean
    @Scope("prototype")
    public PrototypeBean prototypeBean() {
        return new PrototypeBean();
    }
}

// Usage
ApplicationContext context = new AnnotationConfigApplicationContext(ScopeConfig.class);

SingletonBean s1 = context.getBean(SingletonBean.class);
SingletonBean s2 = context.getBean(SingletonBean.class);
System.out.println(s1 == s2);  // true (same instance)

PrototypeBean p1 = context.getBean(PrototypeBean.class);
PrototypeBean p2 = context.getBean(PrototypeBean.class);
System.out.println(p1 == p2);  // false (different instances)
```

### Scoped Proxy

**Problem:** Injecting prototype into singleton

```java
@Service
@Scope("singleton")
public class UserService {
    @Autowired
    private PrototypeBean prototypeBean;  // Always same instance!
}

@Component
@Scope("prototype")
public class PrototypeBean {
    // Should be new instance each time
}
```

**Solution:** Use scoped proxy

```java
@Component
@Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class PrototypeBean {
    // Proxy created, new instance on each method call
}
```

---

## Advanced Topics

### @Lazy Initialization

**Lazy Beans:**
- Created when first accessed
- Useful for expensive beans
- Reduces startup time

```java
@Configuration
public class AppConfig {
    
    @Bean
    @Lazy
    public ExpensiveService expensiveService() {
        // Created only when first accessed
        return new ExpensiveService();
    }
}

// Or at class level
@Service
@Lazy
public class UserService {
    // Entire class is lazy
}
```

### @DependsOn

**Control Bean Creation Order:**

```java
@Configuration
public class AppConfig {
    
    @Bean
    @DependsOn("databaseInitializer")
    public UserService userService() {
        // Created after databaseInitializer
        return new UserService();
    }
    
    @Bean
    public DatabaseInitializer databaseInitializer() {
        return new DatabaseInitializer();
    }
}
```

### Factory Beans

**Custom Bean Creation Logic:**

```java
public class ConnectionFactoryBean implements FactoryBean<Connection> {
    
    private String url;
    private String username;
    private String password;
    
    @Override
    public Connection getObject() throws Exception {
        return DriverManager.getConnection(url, username, password);
    }
    
    @Override
    public Class<?> getObjectType() {
        return Connection.class;
    }
    
    @Override
    public boolean isSingleton() {
        return false;  // New connection each time
    }
}

@Configuration
public class AppConfig {
    @Bean
    public FactoryBean<Connection> connectionFactory() {
        ConnectionFactoryBean factory = new ConnectionFactoryBean();
        factory.setUrl("jdbc:mysql://localhost:3306/mydb");
        factory.setUsername("user");
        factory.setPassword("pass");
        return factory;
    }
}
```

### Environment and Profiles

```java
@Configuration
@Profile("production")
public class ProductionConfig {
    @Bean
    public DataSource dataSource() {
        return new ProductionDataSource();
    }
}

@Configuration
@Profile("development")
public class DevelopmentConfig {
    @Bean
    public DataSource dataSource() {
        return new H2DataSource();
    }
}

// Activate profile
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication app = new SpringApplication(Application.class);
        app.setAdditionalProfiles("production");
        app.run(args);
    }
}
```

### Property Injection

```java
@Configuration
@PropertySource("classpath:application.properties")
public class AppConfig {
    
    @Value("${app.name}")
    private String appName;
    
    @Value("${app.version}")
    private String appVersion;
    
    @Value("${database.url}")
    private String databaseUrl;
    
    @Bean
    public AppInfo appInfo() {
        return new AppInfo(appName, appVersion);
    }
}
```

### SpEL (Spring Expression Language)

```java
@Configuration
public class AppConfig {
    
    @Value("#{systemProperties['user.home']}")
    private String userHome;
    
    @Value("#{T(java.lang.Math).random() * 100}")
    private double randomNumber;
    
    @Value("#{userService.getDefaultUser()}")
    private User defaultUser;
    
    @Value("#{userService.users.size()}")
    private int userCount;
}
```

---

## Best Practices

### 1. Prefer Constructor Injection

```java
// ✅ Good
@Service
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}

// ❌ Bad
@Service
public class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

### 2. Use @ComponentScan Wisely

```java
// ✅ Specific packages
@ComponentScan(basePackages = "com.example.services")

// ❌ Too broad
@ComponentScan  // Scans entire classpath
```

### 3. Avoid Circular Dependencies

```java
// ❌ Bad - Circular dependency
@Service
public class ServiceA {
    private final ServiceB serviceB;
    public ServiceA(ServiceB serviceB) { this.serviceB = serviceB; }
}

@Service
public class ServiceB {
    private final ServiceA serviceA;  // Circular!
    public ServiceB(ServiceA serviceA) { this.serviceA = serviceA; }
}

// ✅ Good - Refactor
@Service
public class ServiceA {
    private final ServiceB serviceB;
    public ServiceA(ServiceB serviceB) { this.serviceB = serviceB; }
}

@Service
public class ServiceB {
    // No dependency on ServiceA
}
```

### 4. Use Appropriate Scopes

```java
// ✅ Singleton for stateless services
@Service  // Default singleton
public class UserService {
    // Stateless, can be singleton
}

// ✅ Prototype for stateful beans
@Component
@Scope("prototype")
public class RequestProcessor {
    private String requestId;  // Stateful
}
```

### 5. Initialize Resources in @PostConstruct

```java
@Service
public class DatabaseService {
    private Connection connection;
    
    @PostConstruct
    public void init() {
        // Initialize expensive resources
        this.connection = createConnection();
    }
    
    @PreDestroy
    public void cleanup() {
        // Cleanup resources
        if (connection != null) {
            connection.close();
        }
    }
}
```

---

## Interview Questions & Answers

### Q1: What is the difference between BeanFactory and ApplicationContext?

**Answer:**
- **BeanFactory**: Basic container, lazy initialization, lightweight
- **ApplicationContext**: Advanced container, eager initialization, additional features (events, internationalization, AOP)

### Q2: What are the different types of Dependency Injection?

**Answer:**
1. **Constructor Injection**: Recommended, immutable, required dependencies
2. **Setter Injection**: Optional dependencies, mutable
3. **Field Injection**: Not recommended, hard to test
4. **Method Injection**: Rare, for special cases

### Q3: Explain the Bean Lifecycle in Spring.

**Answer:**
1. Instantiation
2. Populate properties (DI)
3. BeanNameAware, BeanFactoryAware, ApplicationContextAware
4. BeanPostProcessor.postProcessBeforeInitialization
5. @PostConstruct / InitializingBean.afterPropertiesSet
6. BeanPostProcessor.postProcessAfterInitialization
7. Bean ready
8. @PreDestroy / DisposableBean.destroy

### Q4: What is the difference between @Component, @Service, @Repository, and @Controller?

**Answer:**
- **@Component**: Generic Spring component
- **@Service**: Business logic service layer
- **@Repository**: Data access layer, exception translation
- **@Controller**: Web controller (Spring MVC)
- All are stereotypes, functionally similar but semantically different

### Q5: How do you handle circular dependencies?

**Answer:**
1. **Refactor**: Remove circular dependency (best)
2. **@Lazy**: Lazy initialization breaks cycle
3. **Setter Injection**: Use setter for one dependency
4. **Refactor to use events**: Decouple using Spring events

### Q6: What is the difference between Singleton and Prototype scope?

**Answer:**
- **Singleton**: One instance per container, shared, default scope
- **Prototype**: New instance each time, not managed after creation, no destroy callbacks

### Q7: When would you use @Primary vs @Qualifier?

**Answer:**
- **@Primary**: Mark default bean when multiple beans of same type
- **@Qualifier**: Explicitly specify which bean to inject by name
- Use @Primary for default, @Qualifier for specific selection

### Q8: What is a BeanPostProcessor?

**Answer:**
- Interface for custom modification of bean instances
- Called before and after initialization
- Used for AOP proxies, validation, logging, etc.
- Can modify or wrap beans

---

## Summary

**Key Takeaways:**
1. **IoC**: Framework manages object creation and dependencies
2. **DI**: Dependencies injected, not created by objects
3. **Constructor Injection**: Preferred method for DI
4. **Bean Lifecycle**: Multiple phases with callbacks
5. **Scopes**: Singleton (default), Prototype, Request, Session, etc.
6. **ApplicationContext**: Advanced container with additional features
7. **Best Practices**: Constructor injection, avoid circular dependencies, appropriate scopes

**Complete Coverage:**
- IoC and DI concepts
- Container types and usage
- Bean configuration methods
- Complete bean lifecycle
- All bean scopes
- Advanced topics and best practices
- Interview Q&A

---

**Guide Complete** - Ready for interview preparation!

