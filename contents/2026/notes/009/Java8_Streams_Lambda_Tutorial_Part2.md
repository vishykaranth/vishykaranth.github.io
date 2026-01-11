# Java 8 Streams and Lambda Expressions - In-Depth Tutorial

## Part 2: Stream API Fundamentals

---

## Table of Contents

1. [Introduction to Streams](#1-introduction-to-streams)
2. [Stream Creation](#2-stream-creation)
3. [Intermediate Operations](#3-intermediate-operations)
4. [Terminal Operations](#4-terminal-operations)
5. [Stream Pipeline](#5-stream-pipeline)

---

## 1. Introduction to Streams

### 1.1 What are Streams?

A Stream is a sequence of elements supporting sequential and parallel aggregate operations. Streams are not data structures; they take input from Collections, Arrays, or I/O channels.

### 1.2 Key Characteristics

- **Not a data structure**: Streams don't store elements
- **Functional in nature**: Operations produce results but don't modify the source
- **Lazy evaluation**: Intermediate operations are lazy; terminal operations trigger execution
- **Possibly unbounded**: Streams can represent infinite sequences
- **Consumable**: Elements are visited only once during the stream's lifetime

### 1.3 Stream vs Collection

| Feature | Collection | Stream |
|---------|-----------|--------|
| Storage | Stores data | Doesn't store data |
| Modification | Can modify | Immutable operations |
| Traversal | Multiple times | Single traversal |
| Evaluation | Eager | Lazy |
| External Iteration | Yes | Internal iteration |

### 1.4 Stream Operations Categories

1. **Intermediate Operations**: Return a stream, lazy evaluation
   - `filter()`, `map()`, `flatMap()`, `distinct()`, `sorted()`, etc.

2. **Terminal Operations**: Produce a result or side effect, trigger execution
   - `forEach()`, `collect()`, `reduce()`, `count()`, `anyMatch()`, etc.

---

## 2. Stream Creation

### 2.1 From Collections

```java
// List
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> stream = list.stream();

// Set
Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3));
Stream<Integer> stream = set.stream();

// Map (keys, values, or entries)
Map<String, Integer> map = new HashMap<>();
Stream<String> keyStream = map.keySet().stream();
Stream<Integer> valueStream = map.values().stream();
Stream<Map.Entry<String, Integer>> entryStream = map.entrySet().stream();
```

### 2.2 From Arrays

```java
// Using Arrays.stream()
String[] array = {"a", "b", "c"};
Stream<String> stream = Arrays.stream(array);

// Specific range
Stream<String> rangeStream = Arrays.stream(array, 1, 3);  // "b", "c"

// Primitive arrays
int[] intArray = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(intArray);
```

### 2.3 Using Stream.of()

```java
// From individual elements
Stream<String> stream = Stream.of("a", "b", "c");

// From array
String[] array = {"a", "b", "c"};
Stream<String> stream = Stream.of(array);

// Empty stream
Stream<String> emptyStream = Stream.empty();
```

### 2.4 Using Stream.builder()

```java
Stream<String> stream = Stream.<String>builder()
    .add("a")
    .add("b")
    .add("c")
    .build();
```

### 2.5 Using Stream.generate()

```java
// Infinite stream of random numbers
Stream<Double> randomStream = Stream.generate(Math::random);

// Infinite stream of constant value
Stream<String> constantStream = Stream.generate(() -> "Hello");

// Limited infinite stream
List<Double> randomNumbers = Stream.generate(Math::random)
    .limit(10)
    .collect(Collectors.toList());
```

### 2.6 Using Stream.iterate()

```java
// Infinite stream starting from seed
Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);

// Limited infinite stream
List<Integer> evenNumbers = Stream.iterate(0, n -> n + 2)
    .limit(10)
    .collect(Collectors.toList());  // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

// With predicate (Java 9+)
Stream<Integer> limitedStream = Stream.iterate(0, n -> n < 100, n -> n + 2);
```

### 2.7 From Files

```java
// Reading lines from file
try (Stream<String> lines = Files.lines(Paths.get("file.txt"))) {
    lines.forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

// Reading all lines
List<String> allLines = Files.readAllLines(Paths.get("file.txt"));
Stream<String> stream = allLines.stream();
```

### 2.8 Primitive Streams

```java
// IntStream
IntStream intStream = IntStream.of(1, 2, 3, 4, 5);
IntStream range = IntStream.range(1, 10);        // 1 to 9
IntStream rangeClosed = IntStream.rangeClosed(1, 10);  // 1 to 10

// LongStream
LongStream longStream = LongStream.of(1L, 2L, 3L);

// DoubleStream
DoubleStream doubleStream = DoubleStream.of(1.0, 2.0, 3.0);
```

### 2.9 Parallel Streams

```java
// From collection
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> parallelStream = list.parallelStream();

// Convert sequential to parallel
Stream<String> parallel = list.stream().parallel();

// Check if parallel
boolean isParallel = parallelStream.isParallel();
```

---

## 3. Intermediate Operations

Intermediate operations are lazy and return a new stream. They don't execute until a terminal operation is called.

### 3.1 filter()

Filters elements based on a predicate.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Filter even numbers
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());  // [2, 4, 6, 8, 10]

// Filter strings with length > 3
List<String> words = Arrays.asList("hello", "world", "java", "stream");
List<String> longWords = words.stream()
    .filter(word -> word.length() > 4)
    .collect(Collectors.toList());  // ["hello", "world", "stream"]

// Multiple filters (chaining)
List<Integer> result = numbers.stream()
    .filter(n -> n > 5)
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());  // [6, 8, 10]
```

### 3.2 map()

Transforms each element using a function.

```java
List<String> words = Arrays.asList("hello", "world", "java");

// Transform to uppercase
List<String> upperCase = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());  // ["HELLO", "WORLD", "JAVA"]

// Transform to lengths
List<Integer> lengths = words.stream()
    .map(String::length)
    .collect(Collectors.toList());  // [5, 5, 4]

// Transform to objects
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30)
);
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());  // ["Alice", "Bob"]
```

### 3.3 flatMap()

Flattens nested structures. Each element is mapped to a stream, and all streams are flattened into one.

```java
// Flattening nested lists
List<List<String>> nestedList = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d"),
    Arrays.asList("e", "f")
);

List<String> flattened = nestedList.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());  // ["a", "b", "c", "d", "e", "f"]

// Splitting strings into words
List<String> sentences = Arrays.asList(
    "Hello World",
    "Java Streams",
    "Lambda Expressions"
);

List<String> words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList());  // ["Hello", "World", "Java", "Streams", ...]

// One-to-many mapping
List<Person> people = Arrays.asList(
    new Person("Alice", Arrays.asList("Java", "Python")),
    new Person("Bob", Arrays.asList("C++", "JavaScript"))
);

List<String> allSkills = people.stream()
    .flatMap(person -> person.getSkills().stream())
    .collect(Collectors.toList());  // ["Java", "Python", "C++", "JavaScript"]
```

### 3.4 distinct()

Removes duplicate elements based on equals().

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5);

List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList());  // [1, 2, 3, 4, 5]

// For custom objects, ensure equals() and hashCode() are implemented
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Alice", 25)  // Duplicate
);

List<Person> uniquePeople = people.stream()
    .distinct()
    .collect(Collectors.toList());
```

### 3.5 sorted()

Sorts elements based on natural order or a comparator.

```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9, 3);

// Natural order
List<Integer> sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList());  // [1, 2, 3, 5, 8, 9]

// Reverse order
List<Integer> reverseSorted = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());  // [9, 8, 5, 3, 2, 1]

// Custom comparator
List<String> words = Arrays.asList("hello", "world", "java", "stream");
List<String> sortedByLength = words.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());  // ["java", "hello", "world", "stream"]

// Multiple criteria
List<Person> people = Arrays.asList(
    new Person("Alice", 25),
    new Person("Bob", 30),
    new Person("Charlie", 25)
);

List<Person> sorted = people.stream()
    .sorted(Comparator
        .comparing(Person::getAge)
        .thenComparing(Person::getName))
    .collect(Collectors.toList());
```

### 3.6 peek()

Performs an action on each element as it's consumed. Useful for debugging.

```java
List<String> words = Arrays.asList("hello", "world", "java");

List<String> result = words.stream()
    .filter(word -> word.length() > 4)
    .peek(word -> System.out.println("Filtered: " + word))
    .map(String::toUpperCase)
    .peek(word -> System.out.println("Mapped: " + word))
    .collect(Collectors.toList());
```

### 3.7 limit()

Limits the stream to a maximum number of elements.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> firstFive = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());  // [1, 2, 3, 4, 5]

// With infinite stream
List<Integer> firstTen = Stream.iterate(0, n -> n + 1)
    .limit(10)
    .collect(Collectors.toList());  // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

### 3.8 skip()

Skips the first n elements.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> skipped = numbers.stream()
    .skip(5)
    .collect(Collectors.toList());  // [6, 7, 8, 9, 10]

// Combined with limit
List<Integer> result = numbers.stream()
    .skip(2)
    .limit(5)
    .collect(Collectors.toList());  // [3, 4, 5, 6, 7]
```

### 3.9 takeWhile() and dropWhile() (Java 9+)

```java
// takeWhile: takes elements while predicate is true
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
List<Integer> result = numbers.stream()
    .takeWhile(n -> n < 5)
    .collect(Collectors.toList());  // [1, 2, 3, 4]

// dropWhile: drops elements while predicate is true
List<Integer> result2 = numbers.stream()
    .dropWhile(n -> n < 5)
    .collect(Collectors.toList());  // [5, 6, 7, 8, 9, 10]
```

---

## 4. Terminal Operations

Terminal operations produce a result or side effect and trigger the execution of the stream pipeline.

### 4.1 forEach()

Performs an action for each element.

```java
List<String> words = Arrays.asList("hello", "world", "java");

// Print each element
words.stream()
    .forEach(System.out::println);

// Perform custom action
words.stream()
    .forEach(word -> System.out.println("Word: " + word + ", Length: " + word.length()));
```

### 4.2 collect()

Accumulates elements into a collection or other data structure.

```java
List<String> words = Arrays.asList("hello", "world", "java");

// To List
List<String> list = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// To Set
Set<String> set = words.stream()
    .collect(Collectors.toSet());

// To Map
Map<String, Integer> map = words.stream()
    .collect(Collectors.toMap(
        word -> word,
        String::length
    ));

// To specific collection
LinkedList<String> linkedList = words.stream()
    .collect(Collectors.toCollection(LinkedList::new));
```

### 4.3 reduce()

Reduces the stream to a single value using an accumulator function.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum
Optional<Integer> sum = numbers.stream()
    .reduce((a, b) -> a + b);
// or
Integer sum2 = numbers.stream()
    .reduce(0, (a, b) -> a + b);  // 15

// Product
Integer product = numbers.stream()
    .reduce(1, (a, b) -> a * b);  // 120

// Max
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);

// Min
Optional<Integer> min = numbers.stream()
    .reduce(Integer::min);

// String concatenation
List<String> words = Arrays.asList("Hello", "World", "Java");
String result = words.stream()
    .reduce("", (a, b) -> a + " " + b).trim();  // "Hello World Java"
```

### 4.4 count()

Returns the count of elements in the stream.

```java
List<String> words = Arrays.asList("hello", "world", "java");

long count = words.stream()
    .filter(word -> word.length() > 4)
    .count();  // 2
```

### 4.5 anyMatch(), allMatch(), noneMatch()

Boolean terminal operations.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// anyMatch: true if any element matches
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);  // true

// allMatch: true if all elements match
boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);  // true

// noneMatch: true if no elements match
boolean noNegative = numbers.stream()
    .noneMatch(n -> n < 0);  // true
```

### 4.6 findFirst() and findAny()

Find elements in the stream.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// findFirst: returns first element
Optional<Integer> first = numbers.stream()
    .filter(n -> n > 3)
    .findFirst();  // Optional[4]

// findAny: returns any element (useful for parallel streams)
Optional<Integer> any = numbers.stream()
    .filter(n -> n > 3)
    .findAny();  // Optional[4] or Optional[5] (non-deterministic in parallel)
```

### 4.7 min() and max()

Find minimum and maximum elements.

```java
List<Integer> numbers = Arrays.asList(5, 2, 8, 1, 9, 3);

// Min
Optional<Integer> min = numbers.stream()
    .min(Integer::compareTo);  // Optional[1]

// Max
Optional<Integer> max = numbers.stream()
    .max(Integer::compareTo);  // Optional[9]

// With custom comparator
List<String> words = Arrays.asList("hello", "world", "java");
Optional<String> shortest = words.stream()
    .min(Comparator.comparing(String::length));  // Optional["java"]
```

### 4.8 toArray()

Converts stream to array.

```java
List<String> words = Arrays.asList("hello", "world", "java");

// Object array
String[] array = words.stream()
    .toArray(String[]::new);

// Int array
int[] intArray = IntStream.of(1, 2, 3, 4, 5)
    .toArray();
```

---

## 5. Stream Pipeline

### 5.1 Understanding Stream Pipeline

A stream pipeline consists of:
1. **Source**: Collection, array, generator function, etc.
2. **Intermediate Operations**: Zero or more lazy operations
3. **Terminal Operation**: One operation that triggers execution

```java
List<String> result = list.stream()           // Source
    .filter(s -> s.length() > 5)             // Intermediate (lazy)
    .map(String::toUpperCase)                // Intermediate (lazy)
    .sorted()                                // Intermediate (lazy)
    .collect(Collectors.toList());            // Terminal (triggers execution)
```

### 5.2 Lazy Evaluation

Intermediate operations are lazy - they don't execute until a terminal operation is called.

```java
List<String> words = Arrays.asList("hello", "world", "java", "stream");

// No execution happens here
Stream<String> filtered = words.stream()
    .filter(word -> {
        System.out.println("Filtering: " + word);  // Won't print yet
        return word.length() > 4;
    })
    .map(word -> {
        System.out.println("Mapping: " + word);  // Won't print yet
        return word.toUpperCase();
    });

// Execution happens here
List<String> result = filtered.collect(Collectors.toList());
// Now the println statements will execute
```

### 5.3 Short-Circuiting Operations

Some operations can terminate early without processing all elements.

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// findFirst is short-circuiting
Optional<Integer> firstEven = numbers.stream()
    .filter(n -> {
        System.out.println("Checking: " + n);
        return n % 2 == 0;
    })
    .findFirst();  // Stops after finding 2

// anyMatch is short-circuiting
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);  // Stops after finding first even

// limit is short-circuiting
List<Integer> firstFive = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());  // Only processes first 5
```

### 5.4 Complete Pipeline Example

```java
class Person {
    private String name;
    private int age;
    private String city;
    
    // Constructors, getters, setters
}

List<Person> people = Arrays.asList(
    new Person("Alice", 25, "New York"),
    new Person("Bob", 30, "London"),
    new Person("Charlie", 25, "New York"),
    new Person("David", 35, "Paris"),
    new Person("Eve", 28, "New York")
);

// Complex pipeline
List<String> result = people.stream()
    .filter(person -> person.getAge() >= 25)           // Filter by age
    .filter(person -> person.getCity().equals("New York"))  // Filter by city
    .sorted(Comparator.comparing(Person::getAge))      // Sort by age
    .map(Person::getName)                              // Extract names
    .map(String::toUpperCase)                          // Convert to uppercase
    .distinct()                                        // Remove duplicates
    .collect(Collectors.toList());                     // Collect to list

// Result: ["ALICE", "CHARLIE", "EVE"]
```

### 5.5 Performance Considerations

```java
// Good: Filter first to reduce elements
List<String> result = list.stream()
    .filter(s -> s.length() > 5)      // Reduces elements early
    .map(String::toUpperCase)         // Fewer elements to map
    .collect(Collectors.toList());

// Bad: Map first (processes more elements)
List<String> result = list.stream()
    .map(String::toUpperCase)         // Processes all elements
    .filter(s -> s.length() > 5)     // Then filters
    .collect(Collectors.toList());
```

---

## Summary of Part 2

**Key Takeaways:**
1. **Streams** are sequences of elements supporting functional operations
2. **Stream creation** from collections, arrays, generators, files, etc.
3. **Intermediate operations** are lazy and return new streams
4. **Terminal operations** trigger execution and produce results
5. **Stream pipeline** combines source, intermediate, and terminal operations
6. **Lazy evaluation** means operations execute only when terminal operation is called

**Next**: Part 3 will cover Collectors, grouping, partitioning, and advanced stream operations.

