# Java 8 Streams and Lambda Expressions - In-Depth Tutorial

## Part 4: Real-World Use Cases, Patterns, and Best Practices

---

## Table of Contents

1. [Real-World Use Cases](#1-real-world-use-cases)
2. [Common Patterns](#2-common-patterns)
3. [Performance Optimization](#3-performance-optimization)
4. [Troubleshooting and Debugging](#4-troubleshooting-and-debugging)
5. [Best Practices Summary](#5-best-practices-summary)

---

## 1. Real-World Use Cases

### 1.1 Data Processing and Transformation

#### Use Case: Processing CSV Data

```java
// Read CSV and transform to objects
List<Employee> employees = Files.lines(Paths.get("employees.csv"))
    .skip(1)  // Skip header
    .map(line -> {
        String[] parts = line.split(",");
        return new Employee(
            parts[0],
            Integer.parseInt(parts[1]),
            parts[2],
            Double.parseDouble(parts[3])
        );
    })
    .collect(Collectors.toList());

// Filter and aggregate
Map<String, Double> avgSalaryByDepartment = employees.stream()
    .filter(e -> e.getAge() >= 25)
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));
```

#### Use Case: API Response Processing

```java
// Process API responses
List<ApiResponse> responses = // ... from API calls

// Extract and flatten nested data
List<String> allUserIds = responses.stream()
    .flatMap(response -> response.getUsers().stream())
    .map(User::getId)
    .distinct()
    .collect(Collectors.toList());

// Group by status and count
Map<String, Long> statusCount = responses.stream()
    .collect(Collectors.groupingBy(
        ApiResponse::getStatus,
        Collectors.counting()
    ));
```

### 1.2 Data Validation and Filtering

#### Use Case: Input Validation

```java
class ValidationResult {
    private boolean valid;
    private List<String> errors;
    // Getters, setters
}

List<String> inputs = Arrays.asList("user@example.com", "invalid", "test@test.com");

// Validate emails
List<ValidationResult> results = inputs.stream()
    .map(email -> {
        List<String> errors = new ArrayList<>();
        if (!email.contains("@")) {
            errors.add("Missing @ symbol");
        }
        if (email.length() < 5) {
            errors.add("Email too short");
        }
        return new ValidationResult(errors.isEmpty(), errors);
    })
    .collect(Collectors.toList());

// Separate valid and invalid
Map<Boolean, List<String>> partitioned = inputs.stream()
    .collect(Collectors.partitioningBy(
        email -> email.contains("@") && email.length() >= 5
    ));
```

### 1.3 Data Aggregation and Reporting

#### Use Case: Sales Report Generation

```java
class Sale {
    private LocalDate date;
    private String product;
    private double amount;
    private String region;
    // Getters, setters
}

List<Sale> sales = // ... sales data

// Monthly sales report
Map<YearMonth, Double> monthlySales = sales.stream()
    .collect(Collectors.groupingBy(
        sale -> YearMonth.from(sale.getDate()),
        Collectors.summingDouble(Sale::getAmount)
    ));

// Top products by region
Map<String, List<Map.Entry<String, Double>>> topProductsByRegion = sales.stream()
    .collect(Collectors.groupingBy(
        Sale::getRegion,
        Collectors.groupingBy(
            Sale::getProduct,
            Collectors.summingDouble(Sale::getAmount)
        )
    ))
    .entrySet().stream()
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        entry -> entry.getValue().entrySet().stream()
            .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
            .limit(5)
            .collect(Collectors.toList())
    ));
```

### 1.4 Data Deduplication and Cleanup

#### Use Case: Removing Duplicates

```java
class Transaction {
    private String id;
    private String accountId;
    private double amount;
    private LocalDateTime timestamp;
    // Getters, setters
    
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Transaction that = (Transaction) o;
        return Objects.equals(id, that.id);
    }
    
    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}

List<Transaction> transactions = // ... transactions

// Remove duplicates by ID
List<Transaction> uniqueTransactions = transactions.stream()
    .distinct()
    .collect(Collectors.toList());

// Remove duplicates by account and amount (custom logic)
List<Transaction> uniqueByAccountAndAmount = transactions.stream()
    .collect(Collectors.toMap(
        t -> t.getAccountId() + ":" + t.getAmount(),
        Function.identity(),
        (existing, replacement) -> existing  // Keep first occurrence
    ))
    .values()
    .stream()
    .collect(Collectors.toList());
```

### 1.5 Data Transformation and Mapping

#### Use Case: DTO Mapping

```java
class User {
    private Long id;
    private String firstName;
    private String lastName;
    private String email;
    // Getters, setters
}

class UserDTO {
    private Long id;
    private String fullName;
    private String email;
    // Getters, setters
}

List<User> users = // ... users

// Transform to DTOs
List<UserDTO> userDTOs = users.stream()
    .map(user -> {
        UserDTO dto = new UserDTO();
        dto.setId(user.getId());
        dto.setFullName(user.getFirstName() + " " + user.getLastName());
        dto.setEmail(user.getEmail());
        return dto;
    })
    .collect(Collectors.toList());

// Or using a mapper method
List<UserDTO> userDTOs2 = users.stream()
    .map(this::toUserDTO)
    .collect(Collectors.toList());

private UserDTO toUserDTO(User user) {
    UserDTO dto = new UserDTO();
    dto.setId(user.getId());
    dto.setFullName(user.getFirstName() + " " + user.getLastName());
    dto.setEmail(user.getEmail());
    return dto;
}
```

---

## 2. Common Patterns

### 2.1 Pattern: Transform and Filter

```java
// Common pattern: transform then filter
List<String> result = list.stream()
    .map(String::toUpperCase)
    .filter(s -> s.length() > 5)
    .collect(Collectors.toList());

// Or filter then transform (more efficient if filter reduces elements)
List<String> result2 = list.stream()
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 2.2 Pattern: Find or Default

```java
// Find first matching element or return default
String result = list.stream()
    .filter(s -> s.startsWith("A"))
    .findFirst()
    .orElse("Not Found");

// Find first matching element or throw exception
String result2 = list.stream()
    .filter(s -> s.startsWith("A"))
    .findFirst()
    .orElseThrow(() -> new NotFoundException("No element found"));
```

### 2.3 Pattern: Group and Aggregate

```java
// Group by key and aggregate values
Map<String, Double> result = items.stream()
    .collect(Collectors.groupingBy(
        Item::getCategory,
        Collectors.summingDouble(Item::getPrice)
    ));
```

### 2.4 Pattern: Flatten Nested Collections

```java
// Flatten list of lists
List<List<String>> nested = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d")
);

List<String> flattened = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
```

### 2.5 Pattern: Conditional Collection

```java
// Collect to different collections based on condition
Map<Boolean, List<String>> partitioned = list.stream()
    .collect(Collectors.partitioningBy(s -> s.length() > 5));

List<String> longStrings = partitioned.get(true);
List<String> shortStrings = partitioned.get(false);
```

### 2.6 Pattern: Chaining Optional Operations

```java
// Chain optional operations safely
Optional<String> result = list.stream()
    .filter(s -> s.startsWith("A"))
    .findFirst()
    .map(String::toUpperCase)
    .filter(s -> s.length() > 5);
```

### 2.7 Pattern: Collecting to Map with Conflict Resolution

```java
// Handle duplicate keys when collecting to map
Map<String, String> map = list.stream()
    .collect(Collectors.toMap(
        String::toLowerCase,
        Function.identity(),
        (existing, replacement) -> existing  // Keep first
    ));
```

### 2.8 Pattern: Parallel Processing with Result Combination

```java
// Process in parallel and combine results
List<String> result = list.parallelStream()
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

---

## 3. Performance Optimization

### 3.1 Order of Operations Matters

```java
// Bad: Map all elements, then filter
List<String> result1 = list.stream()
    .map(String::toUpperCase)      // Processes all elements
    .filter(s -> s.length() > 5)   // Then filters
    .collect(Collectors.toList());

// Good: Filter first, then map (fewer elements to process)
List<String> result2 = list.stream()
    .filter(s -> s.length() > 5)   // Reduces elements early
    .map(String::toUpperCase)      // Fewer elements to map
    .collect(Collectors.toList());
```

### 3.2 Use Primitive Streams

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Bad: Boxing overhead
int sum1 = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();

// Good: Use IntStream directly
int sum2 = numbers.stream()
    .mapToInt(i -> i)
    .sum();

// Even better: If you have int array
int[] intArray = {1, 2, 3, 4, 5};
int sum3 = Arrays.stream(intArray).sum();
```

### 3.3 Avoid Unnecessary Operations

```java
// Bad: Unnecessary distinct before limit
List<String> result1 = list.stream()
    .distinct()
    .limit(10)
    .collect(Collectors.toList());

// Good: Limit first if you only need first 10
List<String> result2 = list.stream()
    .limit(10)
    .distinct()
    .collect(Collectors.toList());
```

### 3.4 Use Parallel Streams Wisely

```java
// Good: Large dataset, CPU-intensive, stateless
List<Integer> largeList = IntStream.range(1, 1_000_000)
    .boxed()
    .collect(Collectors.toList());

long sum = largeList.parallelStream()
    .filter(n -> n % 2 == 0)
    .mapToInt(Integer::intValue)
    .sum();

// Bad: Small dataset (overhead > benefit)
List<Integer> smallList = Arrays.asList(1, 2, 3, 4, 5);
long sum2 = smallList.parallelStream()  // Unnecessary overhead
    .mapToInt(Integer::intValue)
    .sum();
```

### 3.5 Cache Intermediate Results When Needed

```java
// If you need to reuse a filtered list multiple times
List<String> filtered = list.stream()
    .filter(s -> s.length() > 5)
    .collect(Collectors.toList());  // Materialize once

// Then reuse
List<String> upper = filtered.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

List<String> lower = filtered.stream()
    .map(String::toLowerCase)
    .collect(Collectors.toList());
```

### 3.6 Avoid Side Effects in Streams

```java
// Bad: Side effect in stream
List<String> result = new ArrayList<>();
list.stream()
    .filter(s -> s.length() > 5)
    .forEach(result::add);  // Side effect

// Good: Use collect
List<String> result2 = list.stream()
    .filter(s -> s.length() > 5)
    .collect(Collectors.toList());
```

### 3.7 Use Specialized Collectors

```java
// Good: Use specialized collectors for better performance
long count = list.stream()
    .collect(Collectors.counting());

// Instead of
long count2 = list.stream()
    .map(s -> 1L)
    .reduce(0L, Long::sum);
```

---

## 4. Troubleshooting and Debugging

### 4.1 Using peek() for Debugging

```java
// Debug stream pipeline
List<String> result = list.stream()
    .peek(s -> System.out.println("Original: " + s))
    .filter(s -> s.length() > 5)
    .peek(s -> System.out.println("After filter: " + s))
    .map(String::toUpperCase)
    .peek(s -> System.out.println("After map: " + s))
    .collect(Collectors.toList());
```

### 4.2 Common Errors and Solutions

#### Error: NullPointerException

```java
// Problem: Null elements in stream
List<String> list = Arrays.asList("hello", null, "world");

// Solution 1: Filter nulls
List<String> result1 = list.stream()
    .filter(Objects::nonNull)
    .collect(Collectors.toList());

// Solution 2: Use Optional
List<String> result2 = list.stream()
    .map(Optional::ofNullable)
    .filter(Optional::isPresent)
    .map(Optional::get)
    .collect(Collectors.toList());
```

#### Error: IllegalStateException (Stream already operated upon)

```java
// Problem: Reusing stream
Stream<String> stream = list.stream();
List<String> result1 = stream.filter(s -> s.length() > 5).collect(Collectors.toList());
List<String> result2 = stream.map(String::toUpperCase).collect(Collectors.toList());  // Error!

// Solution: Create new stream for each operation
List<String> result1 = list.stream()
    .filter(s -> s.length() > 5)
    .collect(Collectors.toList());

List<String> result2 = list.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

#### Error: ConcurrentModificationException

```java
// Problem: Modifying collection while iterating
List<String> list = new ArrayList<>(Arrays.asList("a", "b", "c"));
list.stream()
    .forEach(s -> list.add(s.toUpperCase()));  // Error!

// Solution: Collect to new list
List<String> result = list.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 4.3 Performance Debugging

```java
// Measure stream performance
long start = System.nanoTime();
List<String> result = list.stream()
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
long duration = System.nanoTime() - start;
System.out.println("Time: " + duration + " ns");
```

### 4.4 Debugging Parallel Streams

```java
// Add logging to see parallel execution
List<String> result = list.parallelStream()
    .peek(s -> System.out.println(
        Thread.currentThread().getName() + ": " + s
    ))
    .filter(s -> s.length() > 5)
    .collect(Collectors.toList());
```

---

## 5. Best Practices Summary

### 5.1 Code Readability

1. **Use method references when possible**
   ```java
   // Good
   .map(String::toUpperCase)
   
   // Avoid
   .map(s -> s.toUpperCase())
   ```

2. **Keep lambdas short and focused**
   ```java
   // Good: Extract complex logic
   .map(this::complexTransformation)
   
   // Avoid: Long lambda
   .map(s -> {
       // 20 lines of code
   })
   ```

3. **Use descriptive variable names**
   ```java
   // Good
   .filter(word -> word.length() > 5)
   
   // Avoid
   .filter(x -> x.length() > 5)
   ```

### 5.2 Performance

1. **Filter early, map late**
2. **Use primitive streams for primitives**
3. **Avoid unnecessary operations**
4. **Use parallel streams for large datasets**
5. **Cache intermediate results if reused**

### 5.3 Correctness

1. **Avoid side effects in streams**
2. **Handle nulls appropriately**
3. **Don't reuse streams**
4. **Use appropriate collectors**
5. **Handle exceptions properly**

### 5.4 Maintainability

1. **Extract complex logic to methods**
2. **Use meaningful names**
3. **Add comments for complex operations**
4. **Test stream operations thoroughly**
5. **Consider readability over brevity**

---

## Complete Example: E-Commerce Order Processing

```java
class Order {
    private String orderId;
    private String customerId;
    private LocalDate orderDate;
    private List<OrderItem> items;
    private OrderStatus status;
    // Getters, setters
}

class OrderItem {
    private String productId;
    private int quantity;
    private double price;
    // Getters, setters
}

enum OrderStatus {
    PENDING, PROCESSING, SHIPPED, DELIVERED, CANCELLED
}

List<Order> orders = // ... orders

// 1. Calculate total revenue
double totalRevenue = orders.stream()
    .filter(order -> order.getStatus() != OrderStatus.CANCELLED)
    .flatMap(order -> order.getItems().stream())
    .mapToDouble(item -> item.getQuantity() * item.getPrice())
    .sum();

// 2. Find top 10 customers by revenue
List<Map.Entry<String, Double>> topCustomers = orders.stream()
    .filter(order -> order.getStatus() != OrderStatus.CANCELLED)
    .collect(Collectors.groupingBy(
        Order::getCustomerId,
        Collectors.summingDouble(order ->
            order.getItems().stream()
                .mapToDouble(item -> item.getQuantity() * item.getPrice())
                .sum()
        )
    ))
    .entrySet().stream()
    .sorted(Map.Entry.<String, Double>comparingByValue().reversed())
    .limit(10)
    .collect(Collectors.toList());

// 3. Monthly sales report
Map<YearMonth, Map<String, Double>> monthlySalesByProduct = orders.stream()
    .filter(order -> order.getStatus() != OrderStatus.CANCELLED)
    .flatMap(order -> order.getItems().stream()
        .map(item -> new AbstractMap.SimpleEntry<>(
            YearMonth.from(order.getOrderDate()),
            item
        ))
    )
    .collect(Collectors.groupingBy(
        Map.Entry::getKey,
        Collectors.groupingBy(
            entry -> entry.getValue().getProductId(),
            Collectors.summingDouble(
                entry -> entry.getValue().getQuantity() * entry.getValue().getPrice()
            )
        )
    ));

// 4. Find orders needing attention (pending > 7 days)
List<Order> ordersNeedingAttention = orders.stream()
    .filter(order -> order.getStatus() == OrderStatus.PENDING)
    .filter(order -> ChronoUnit.DAYS.between(
        order.getOrderDate(),
        LocalDate.now()
    ) > 7)
    .collect(Collectors.toList());

// 5. Product popularity ranking
Map<String, Long> productPopularity = orders.stream()
    .filter(order -> order.getStatus() != OrderStatus.CANCELLED)
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.groupingBy(
        OrderItem::getProductId,
        Collectors.summingLong(OrderItem::getQuantity)
    ));

List<Map.Entry<String, Long>> topProducts = productPopularity.entrySet().stream()
    .sorted(Map.Entry.<String, Long>comparingByValue().reversed())
    .limit(20)
    .collect(Collectors.toList());
```

---

## Summary of Part 4

**Key Takeaways:**
1. **Real-world use cases** demonstrate practical applications
2. **Common patterns** provide reusable solutions
3. **Performance optimization** improves efficiency
4. **Troubleshooting** helps debug issues
5. **Best practices** ensure maintainable code

**Complete Tutorial Summary:**
- **Part 1**: Lambda expressions, functional interfaces, method references
- **Part 2**: Stream API fundamentals, intermediate and terminal operations
- **Part 3**: Collectors, grouping, partitioning, parallel streams
- **Part 4**: Real-world use cases, patterns, optimization, best practices

This comprehensive tutorial covers all aspects of Java 8 Streams and Lambda expressions, from basics to advanced usage patterns. Practice with these examples and adapt them to your specific use cases!

