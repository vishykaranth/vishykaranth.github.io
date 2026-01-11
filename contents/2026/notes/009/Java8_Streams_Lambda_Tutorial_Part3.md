# Java 8 Streams and Lambda Expressions - In-Depth Tutorial

## Part 3: Collectors and Advanced Stream Operations

---

## Table of Contents

1. [Collectors Overview](#1-collectors-overview)
2. [Basic Collectors](#2-basic-collectors)
3. [Grouping and Partitioning](#3-grouping-and-partitioning)
4. [Custom Collectors](#4-custom-collectors)
5. [Parallel Streams](#5-parallel-streams)
6. [Advanced Examples](#6-advanced-examples)

---

## 1. Collectors Overview

### 1.1 What are Collectors?

Collectors are implementations of the `Collector` interface that provide various useful reduction operations, such as accumulating elements into collections, summarizing elements according to various criteria, etc.

### 1.2 Collector Interface

```java
public interface Collector<T, A, R> {
    Supplier<A> supplier();
    BiConsumer<A, T> accumulator();
    BinaryOperator<A> combiner();
    Function<A, R> finisher();
    Set<Characteristics> characteristics();
}
```

### 1.3 Using Collectors

```java
import java.util.stream.Collectors;

List<String> words = Arrays.asList("hello", "world", "java");
List<String> result = words.stream()
    .collect(Collectors.toList());
```

---

## 2. Basic Collectors

### 2.1 toList(), toSet(), toCollection()

```java
List<String> words = Arrays.asList("hello", "world", "java", "hello");

// To List
List<String> list = words.stream()
    .collect(Collectors.toList());

// To Set (removes duplicates)
Set<String> set = words.stream()
    .collect(Collectors.toSet());  // ["hello", "world", "java"]

// To specific collection
LinkedList<String> linkedList = words.stream()
    .collect(Collectors.toCollection(LinkedList::new));

TreeSet<String> treeSet = words.stream()
    .collect(Collectors.toCollection(TreeSet::new));
```

### 2.2 toMap()

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Charlie", 25)
);

// Simple toMap: key -> value
Map<String, Integer> nameToAge = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge
    ));
// Result: {"Alice": 25, "Bob": 30, "Charlie": 25}

// With merge function (handles duplicates)
List<String> words = Arrays.asList("hello", "world", "hello");
Map<String, Integer> wordCount = words.stream()
    .collect(Collectors.toMap(
        word -> word,
        word -> 1,
        (existing, replacement) -> existing + 1
    ));
// Result: {"hello": 2, "world": 1}

// With map supplier (specific map type)
TreeMap<String, Integer> treeMap = people.stream()
    .collect(Collectors.toMap(
        Person::getName,
        Person::getAge,
        (existing, replacement) -> existing,
        TreeMap::new
    ));
```

### 2.3 joining()

Concatenates input elements into a String.

```java
List<String> words = Arrays.asList("hello", "world", "java");

// Simple joining
String result = words.stream()
    .collect(Collectors.joining());  // "helloworldjava"

// With delimiter
String result2 = words.stream()
    .collect(Collectors.joining(", "));  // "hello, world, java"

// With prefix and suffix
String result3 = words.stream()
    .collect(Collectors.joining(", ", "[", "]"));  // "[hello, world, java]"
```

### 2.4 counting()

Counts the number of elements.

```java
List<String> words = Arrays.asList("hello", "world", "java");

Long count = words.stream()
    .filter(word -> word.length() > 4)
    .collect(Collectors.counting());  // 2

// Equivalent to
long count2 = words.stream()
    .filter(word -> word.length() > 4)
    .count();
```

### 2.5 summarizingInt(), summarizingLong(), summarizingDouble()

Provides statistics about numeric elements.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

IntSummaryStatistics stats = numbers.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));

System.out.println("Count: " + stats.getCount());        // 10
System.out.println("Sum: " + stats.getSum());            // 55
System.out.println("Min: " + stats.getMin());            // 1
System.out.println("Max: " + stats.getMax());            // 10
System.out.println("Average: " + stats.getAverage());    // 5.5
```

### 2.6 summingInt(), summingLong(), summingDouble()

Sums numeric elements.

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Charlie", 25)
);

Integer totalAge = people.stream()
    .collect(Collectors.summingInt(Person::getAge));  // 80

// Equivalent to
int totalAge2 = people.stream()
    .mapToInt(Person::getAge)
    .sum();
```

### 2.7 averagingInt(), averagingLong(), averagingDouble()

Calculates average of numeric elements.

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Charlie", 25)
);

Double averageAge = people.stream()
    .collect(Collectors.averagingInt(Person::getAge));  // 26.666...

// Equivalent to
OptionalDouble averageAge2 = people.stream()
    .mapToInt(Person::getAge)
    .average();
```

### 2.8 maxBy() and minBy()

Finds maximum and minimum elements.

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Charlie", 25)
);

// Max by age
Optional<Person> oldest = people.stream()
    .collect(Collectors.maxBy(Comparator.comparing(Person::getAge)));

// Min by age
Optional<Person> youngest = people.stream()
    .collect(Collectors.minBy(Comparator.comparing(Person::getAge)));

// Max by name (alphabetically)
Optional<Person> lastByName = people.stream()
    .collect(Collectors.maxBy(Comparator.comparing(Person::getName)));
```

### 2.9 reducing()

General-purpose reduction operation.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum
Optional<Integer> sum = numbers.stream()
    .collect(Collectors.reducing((a, b) -> a + b));  // Optional[15]

// Sum with identity
Integer sum2 = numbers.stream()
    .collect(Collectors.reducing(0, (a, b) -> a + b));  // 15

// Sum with mapper
Integer sumOfSquares = numbers.stream()
    .collect(Collectors.reducing(
        0,
        n -> n * n,
        (a, b) -> a + b
    ));  // 55 (1² + 2² + 3² + 4² + 5²)
```

---

## 3. Grouping and Partitioning

### 3.1 groupingBy()

Groups elements by a classification function.

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25, "New York"),
    new Person("Bob", 30, "London"),
    new Person("Charlie", 25, "New York"),
    new Person("David", 35, "Paris"),
    new Person("Eve", 28, "New York")
);

// Simple grouping
Map<String, List<Person>> byCity = people.stream()
    .collect(Collectors.groupingBy(Person::getCity));
// Result:
// {
//   "New York": [Alice, Charlie, Eve],
//   "London": [Bob],
//   "Paris": [David]
// }

// Grouping with downstream collector
Map<String, Long> countByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.counting()
    ));
// Result: {"New York": 3, "London": 1, "Paris": 1}

// Grouping with custom map type
TreeMap<String, List<Person>> sortedByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        TreeMap::new,
        Collectors.toList()
    ));

// Grouping with multiple downstream collectors
Map<String, IntSummaryStatistics> ageStatsByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.summarizingInt(Person::getAge)
    ));
```

### 3.2 partitioningBy()

Partitions elements into two groups based on a predicate.

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Charlie", 25),
    new Person("David", 35)
);

// Simple partitioning
Map<Boolean, List<Person>> byAge = people.stream()
    .collect(Collectors.partitioningBy(p -> p.getAge() >= 30));
// Result:
// {
//   true: [Bob, David],
//   false: [Alice, Charlie]
// }

// Partitioning with downstream collector
Map<Boolean, Long> countByAge = people.stream()
    .collect(Collectors.partitioningBy(
        p -> p.getAge() >= 30,
        Collectors.counting()
    ));
// Result: {true: 2, false: 2}

// Partitioning with multiple downstream collectors
Map<Boolean, IntSummaryStatistics> ageStats = people.stream()
    .collect(Collectors.partitioningBy(
        p -> p.getAge() >= 30,
        Collectors.summarizingInt(Person::getAge)
    ));
```

### 3.3 Multi-level Grouping

```java
List<Person> people = Arrays.asList(
    new Person("Alice", 25, "New York", "Engineer"),
    new Person("Bob", 30, "London", "Manager"),
    new Person("Charlie", 25, "New York", "Engineer"),
    new Person("David", 35, "Paris", "Manager")
);

// Group by city, then by job
Map<String, Map<String, List<Person>>> byCityAndJob = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.groupingBy(Person::getJob)
    ));
// Result:
// {
//   "New York": {
//     "Engineer": [Alice, Charlie]
//   },
//   "London": {
//     "Manager": [Bob]
//   },
//   "Paris": {
//     "Manager": [David]
//   }
// }
```

### 3.4 Advanced Grouping Examples

```java
// Group by age range
Map<String, List<Person>> byAgeRange = people.stream()
    .collect(Collectors.groupingBy(p -> {
        int age = p.getAge();
        if (age < 30) return "Young";
        if (age < 40) return "Middle";
        return "Senior";
    }));

// Group and collect to Set (remove duplicates)
Map<String, Set<Person>> uniqueByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.toSet()
    ));

// Group and get average
Map<String, Double> avgAgeByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.averagingInt(Person::getAge)
    ));

// Group and get max
Map<String, Optional<Person>> oldestByCity = people.stream()
    .collect(Collectors.groupingBy(
        Person::getCity,
        Collectors.maxBy(Comparator.comparing(Person::getAge))
    ));
```

---

## 4. Custom Collectors

### 4.1 Creating Custom Collectors

```java
// Custom collector to collect to comma-separated string with brackets
Collector<String, StringBuilder, String> customCollector = Collector.of(
    StringBuilder::new,                    // Supplier
    (sb, s) -> {                           // Accumulator
        if (sb.length() > 0) sb.append(", ");
        sb.append(s);
    },
    (sb1, sb2) -> {                       // Combiner (for parallel)
        if (sb1.length() > 0 && sb2.length() > 0) {
            sb1.append(", ");
        }
        return sb1.append(sb2);
    },
    sb -> "[" + sb.toString() + "]",      // Finisher
    Collector.Characteristics.UNORDERED  // Characteristics
);

List<String> words = Arrays.asList("hello", "world", "java");
String result = words.stream()
    .collect(customCollector);  // "[hello, world, java]"
```

### 4.2 Collector.of() Example

```java
// Collector to find second largest element
public static <T extends Comparable<T>> Collector<T, ?, Optional<T>> secondLargest() {
    return Collector.of(
        () -> new ArrayList<T>(2),        // Supplier: store top 2
        (list, element) -> {              // Accumulator
            if (list.size() < 2) {
                list.add(element);
                list.sort(Collections.reverseOrder());
            } else if (element.compareTo(list.get(1)) > 0) {
                list.set(1, element);
                list.sort(Collections.reverseOrder());
            }
        },
        (list1, list2) -> {               // Combiner
            list1.addAll(list2);
            list1.sort(Collections.reverseOrder());
            return new ArrayList<>(list1.subList(0, Math.min(2, list1.size())));
        },
        list -> list.size() >= 2 ? Optional.of(list.get(1)) : Optional.empty()
    );
}

List<Integer> numbers = Arrays.asList(1, 5, 3, 8, 2, 9, 4);
Optional<Integer> secondLargest = numbers.stream()
    .collect(secondLargest());  // Optional[8]
```

---

## 5. Parallel Streams

### 5.1 Creating Parallel Streams

```java
List<String> words = Arrays.asList("hello", "world", "java", "stream");

// From collection
Stream<String> parallelStream = words.parallelStream();

// Convert sequential to parallel
Stream<String> parallel = words.stream().parallel();

// Check if parallel
boolean isParallel = parallelStream.isParallel();
```

### 5.2 When to Use Parallel Streams

**Good for:**
- Large datasets
- CPU-intensive operations
- Stateless operations
- Independent operations

**Not good for:**
- Small datasets (overhead > benefit)
- Operations with shared mutable state
- I/O-bound operations
- Operations requiring order

### 5.3 Parallel Stream Examples

```java
List<Integer> numbers = IntStream.range(1, 1_000_000)
    .boxed()
    .collect(Collectors.toList());

// Sequential
long start = System.currentTimeMillis();
long sum1 = numbers.stream()
    .mapToInt(Integer::intValue)
    .sum();
long sequentialTime = System.currentTimeMillis() - start;

// Parallel
start = System.currentTimeMillis();
long sum2 = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum();
long parallelTime = System.currentTimeMillis() - start;

System.out.println("Sequential: " + sequentialTime + "ms");
System.out.println("Parallel: " + parallelTime + "ms");
```

### 5.4 Thread Safety Considerations

```java
// Bad: Shared mutable state
List<Integer> result = new ArrayList<>();
numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .forEach(result::add);  // Not thread-safe!

// Good: Use collect
List<Integer> result2 = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());  // Thread-safe
```

### 5.5 Ordering in Parallel Streams

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Unordered parallel stream (faster)
List<Integer> result1 = numbers.parallelStream()
    .unordered()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// Ordered parallel stream (slower, maintains order)
List<Integer> result2 = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

---

## 6. Advanced Examples

### 6.1 Complex Data Transformation

```java
class Order {
    private String customerId;
    private List<OrderItem> items;
    private LocalDate orderDate;
    // Getters, setters
}

class OrderItem {
    private String productId;
    private int quantity;
    private double price;
    // Getters, setters
}

List<Order> orders = // ... orders

// Find total revenue by customer
Map<String, Double> revenueByCustomer = orders.stream()
    .collect(Collectors.groupingBy(
        Order::getCustomerId,
        Collectors.summingDouble(order ->
            order.getItems().stream()
                .mapToDouble(item -> item.getQuantity() * item.getPrice())
                .sum()
        )
    ));

// Find top 5 customers by revenue
List<Map.Entry<String, Double>> topCustomers = orders.stream()
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
    .limit(5)
    .collect(Collectors.toList());
```

### 6.2 Flattening and Aggregating

```java
class Department {
    private String name;
    private List<Employee> employees;
    // Getters, setters
}

class Employee {
    private String name;
    private double salary;
    // Getters, setters
}

List<Department> departments = // ... departments

// Get all employees from all departments
List<Employee> allEmployees = departments.stream()
    .flatMap(dept -> dept.getEmployees().stream())
    .collect(Collectors.toList());

// Get total salary across all departments
double totalSalary = departments.stream()
    .flatMap(dept -> dept.getEmployees().stream())
    .mapToDouble(Employee::getSalary)
    .sum();

// Get average salary by department
Map<String, Double> avgSalaryByDept = departments.stream()
    .collect(Collectors.toMap(
        Department::getName,
        dept -> dept.getEmployees().stream()
            .mapToDouble(Employee::getSalary)
            .average()
            .orElse(0.0)
    ));
```

### 6.3 Finding and Filtering

```java
List<Person> people = // ... people

// Find first person older than 30
Optional<Person> firstOlderThan30 = people.stream()
    .filter(p -> p.getAge() > 30)
    .findFirst();

// Check if all people are adults
boolean allAdults = people.stream()
    .allMatch(p -> p.getAge() >= 18);

// Check if any person is from New York
boolean hasNewYorker = people.stream()
    .anyMatch(p -> "New York".equals(p.getCity()));

// Get distinct cities
List<String> distinctCities = people.stream()
    .map(Person::getCity)
    .distinct()
    .collect(Collectors.toList());
```

### 6.4 Complex Grouping and Aggregation

```java
// Group by multiple criteria and aggregate
Map<String, Map<String, Double>> salesByRegionAndProduct = orders.stream()
    .flatMap(order -> order.getItems().stream()
        .map(item -> new AbstractMap.SimpleEntry<>(
            order.getRegion(),
            item
        ))
    )
    .collect(Collectors.groupingBy(
        entry -> entry.getKey(),  // Region
        Collectors.groupingBy(
            entry -> entry.getValue().getProductId(),  // Product
            Collectors.summingDouble(
                entry -> entry.getValue().getQuantity() * entry.getValue().getPrice()
            )
        )
    ));
```

### 6.5 Stream Operations with Optional

```java
List<Person> people = // ... people

// Find person by name, then get their age
Optional<Integer> age = people.stream()
    .filter(p -> "Alice".equals(p.getName()))
    .findFirst()
    .map(Person::getAge);

// Chain optional operations
Optional<String> city = people.stream()
    .filter(p -> p.getAge() > 30)
    .findFirst()
    .map(Person::getCity)
    .filter(c -> c.startsWith("New"))
    .map(String::toUpperCase);
```

### 6.6 Combining Multiple Streams

```java
List<String> list1 = Arrays.asList("a", "b", "c");
List<String> list2 = Arrays.asList("d", "e", "f");

// Concatenate streams
List<String> combined = Stream.concat(list1.stream(), list2.stream())
    .collect(Collectors.toList());  // [a, b, c, d, e, f]

// Zip streams (using third-party library or custom implementation)
// Note: Java doesn't have built-in zip, but you can use libraries like Guava
```

### 6.7 Stream with Exception Handling

```java
// Handling checked exceptions in streams
List<String> fileNames = Arrays.asList("file1.txt", "file2.txt", "file3.txt");

// Option 1: Wrap in try-catch
List<String> contents = fileNames.stream()
    .map(filename -> {
        try {
            return Files.readString(Paths.get(filename));
        } catch (IOException e) {
            return "";
        }
    })
    .collect(Collectors.toList());

// Option 2: Extract to method
List<String> contents2 = fileNames.stream()
    .map(this::readFileSafely)
    .collect(Collectors.toList());

private String readFileSafely(String filename) {
    try {
        return Files.readString(Paths.get(filename));
    } catch (IOException e) {
        return "";
    }
}
```

---

## Summary of Part 3

**Key Takeaways:**
1. **Collectors** provide powerful reduction operations
2. **Grouping** organizes elements by classification function
3. **Partitioning** splits elements into two groups
4. **Custom collectors** can be created for specific needs
5. **Parallel streams** can improve performance for large datasets
6. **Advanced patterns** combine multiple operations for complex transformations

**Next**: Part 4 will cover real-world use cases, common patterns, performance optimization, and troubleshooting.

