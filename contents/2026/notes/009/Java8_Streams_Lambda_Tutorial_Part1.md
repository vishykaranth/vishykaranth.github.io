# Java 8 Streams and Lambda Expressions - In-Depth Tutorial

## Part 1: Lambda Expressions and Functional Interfaces

---

## Table of Contents

1. [Introduction to Lambda Expressions](#1-introduction-to-lambda-expressions)
2. [Functional Interfaces](#2-functional-interfaces)
3. [Method References](#3-method-references)
4. [Built-in Functional Interfaces](#4-built-in-functional-interfaces)
5. [Lambda Best Practices](#5-lambda-best-practices)

---

## 1. Introduction to Lambda Expressions

### 1.1 What are Lambda Expressions?

Lambda expressions are anonymous functions that provide a concise way to represent a method interface using an expression. They enable functional programming in Java and make code more readable and expressive.

### 1.2 Syntax

```java
// Basic syntax
(parameters) -> expression

// With body
(parameters) -> { statements; }
```

### 1.3 Evolution: Before and After Lambda

#### Before Java 8 (Anonymous Inner Class)

```java
// Example: Sorting a list
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// Using anonymous inner class
Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
});
```

#### After Java 8 (Lambda Expression)

```java
// Using lambda expression
Collections.sort(names, (s1, s2) -> s1.length() - s2.length());

// Or even better with method reference
names.sort(Comparator.comparing(String::length));
```

### 1.4 Lambda Expression Examples

#### Example 1: Simple Lambda

```java
// Runnable interface
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello World");
    }
};

// Lambda equivalent
Runnable r2 = () -> System.out.println("Hello World");
```

#### Example 2: Lambda with Parameters

```java
// Comparator interface
Comparator<Integer> comp1 = new Comparator<Integer>() {
    @Override
    public int compare(Integer a, Integer b) {
        return a.compareTo(b);
    }
};

// Lambda equivalent
Comparator<Integer> comp2 = (a, b) -> a.compareTo(b);
```

#### Example 3: Lambda with Multiple Statements

```java
// Consumer interface
Consumer<String> consumer1 = new Consumer<String>() {
    @Override
    public void accept(String s) {
        System.out.println("Processing: " + s);
        System.out.println("Length: " + s.length());
    }
};

// Lambda equivalent
Consumer<String> consumer2 = s -> {
    System.out.println("Processing: " + s);
    System.out.println("Length: " + s.length());
};
```

### 1.5 Lambda Expression Types

#### Type 1: No Parameters

```java
Supplier<String> supplier = () -> "Hello";
Runnable runnable = () -> System.out.println("Running");
```

#### Type 2: Single Parameter

```java
Consumer<String> consumer = s -> System.out.println(s);
Function<Integer, Integer> square = x -> x * x;
Predicate<String> isEmpty = s -> s.isEmpty();
```

#### Type 3: Multiple Parameters

```java
Comparator<Integer> comparator = (a, b) -> a.compareTo(b);
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
```

#### Type 4: Explicit Type Declaration

```java
// Type inference (preferred)
Function<String, Integer> length1 = s -> s.length();

// Explicit types (when needed)
Function<String, Integer> length2 = (String s) -> s.length();
BiFunction<Integer, Integer, Integer> multiply = (Integer a, Integer b) -> a * b;
```

---

## 2. Functional Interfaces

### 2.1 What is a Functional Interface?

A functional interface is an interface that contains exactly one abstract method. It can have multiple default or static methods, but only one abstract method.

### 2.2 @FunctionalInterface Annotation

```java
@FunctionalInterface
public interface MyFunctionalInterface {
    void execute();
    
    // Default methods are allowed
    default void defaultMethod() {
        System.out.println("Default method");
    }
    
    // Static methods are allowed
    static void staticMethod() {
        System.out.println("Static method");
    }
}
```

### 2.3 Creating Custom Functional Interfaces

#### Example 1: Simple Functional Interface

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// Usage
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

System.out.println(add.calculate(5, 3));        // 8
System.out.println(multiply.calculate(5, 3));   // 15
```

#### Example 2: Functional Interface with Generic Types

```java
@FunctionalInterface
interface Transformer<T, R> {
    R transform(T input);
}

// Usage
Transformer<String, Integer> stringLength = s -> s.length();
Transformer<Integer, String> intToString = i -> String.valueOf(i);

System.out.println(stringLength.transform("Hello"));  // 5
System.out.println(intToString.transform(123));       // "123"
```

#### Example 3: Functional Interface with Exception

```java
@FunctionalInterface
interface FileProcessor {
    void process(String filename) throws IOException;
}

// Usage
FileProcessor processor = filename -> {
    Files.readAllLines(Paths.get(filename));
    System.out.println("Processed: " + filename);
};
```

---

## 3. Method References

### 3.1 What are Method References?

Method references are a shorthand notation for lambda expressions that call existing methods. They make code more readable when a lambda expression just calls an existing method.

### 3.2 Types of Method References

#### Type 1: Reference to Static Method

```java
// Lambda
Function<String, Integer> parseInt1 = s -> Integer.parseInt(s);

// Method reference
Function<String, Integer> parseInt2 = Integer::parseInt;

// Usage
Integer result = parseInt2.apply("123");  // 123
```

**More Examples:**

```java
// Math operations
Function<Double, Double> sqrt = Math::sqrt;
Function<Double, Double> abs = Math::abs;

// String operations
Function<String, String> upperCase = String::toUpperCase;
Function<String, Integer> length = String::length;
```

#### Type 2: Reference to Instance Method of Particular Object

```java
String prefix = "Hello ";

// Lambda
Function<String, String> addPrefix1 = s -> prefix.concat(s);

// Method reference
Function<String, String> addPrefix2 = prefix::concat;

// Usage
String result = addPrefix2.apply("World");  // "Hello World"
```

**More Examples:**

```java
StringBuilder sb = new StringBuilder();
Consumer<String> append = sb::append;

append.accept("Hello");
append.accept(" World");
System.out.println(sb.toString());  // "Hello World"
```

#### Type 3: Reference to Instance Method of Arbitrary Object

```java
// Lambda
Function<String, String> toUpper1 = s -> s.toUpperCase();

// Method reference
Function<String, String> toUpper2 = String::toUpperCase;

// Usage
String result = toUpper2.apply("hello");  // "HELLO"
```

**More Examples:**

```java
// String methods
Function<String, Integer> length = String::length;
Function<String, String> trim = String::trim;
Predicate<String> isEmpty = String::isEmpty;

// List methods
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(System.out::println);  // Prints each name
```

#### Type 4: Reference to Constructor

```java
// Lambda
Supplier<List<String>> listSupplier1 = () -> new ArrayList<>();

// Constructor reference
Supplier<List<String>> listSupplier2 = ArrayList::new;

// Usage
List<String> list = listSupplier2.get();
```

**More Examples:**

```java
// Constructor with parameters
Function<String, StringBuilder> sbCreator = StringBuilder::new;
BiFunction<String, String, String> concat = String::new;  // Not common

// Array constructor
IntFunction<int[]> arrayCreator = int[]::new;
int[] array = arrayCreator.apply(10);  // Creates array of size 10
```

### 3.3 Method Reference Examples

#### Example 1: Sorting with Method Reference

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "David");

// Lambda
names.sort((s1, s2) -> s1.compareToIgnoreCase(s2));

// Method reference
names.sort(String::compareToIgnoreCase);
```

#### Example 2: Filtering with Method Reference

```java
List<String> strings = Arrays.asList("", "hello", "", "world", "");

// Lambda
List<String> nonEmpty1 = strings.stream()
    .filter(s -> !s.isEmpty())
    .collect(Collectors.toList());

// Method reference (using negation)
List<String> nonEmpty2 = strings.stream()
    .filter(((Predicate<String>) String::isEmpty).negate())
    .collect(Collectors.toList());
```

#### Example 3: Mapping with Method Reference

```java
List<String> words = Arrays.asList("hello", "world", "java");

// Lambda
List<Integer> lengths1 = words.stream()
    .map(s -> s.length())
    .collect(Collectors.toList());

// Method reference
List<Integer> lengths2 = words.stream()
    .map(String::length)
    .collect(Collectors.toList());
```

---

## 4. Built-in Functional Interfaces

### 4.1 Core Functional Interfaces in java.util.function

Java 8 provides a comprehensive set of functional interfaces in the `java.util.function` package.

### 4.2 Function<T, R>

Represents a function that accepts one argument and produces a result.

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    
    // Default methods
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after);
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before);
    static <T> Function<T, T> identity();
}
```

**Examples:**

```java
// Basic usage
Function<String, Integer> length = s -> s.length();
Integer result = length.apply("Hello");  // 5

// Chaining with andThen
Function<String, Integer> length = String::length;
Function<Integer, Integer> square = x -> x * x;
Function<String, Integer> lengthSquared = length.andThen(square);
Integer result = lengthSquared.apply("Hello");  // 25

// Chaining with compose
Function<Integer, Integer> multiply = x -> x * 2;
Function<Integer, Integer> add = x -> x + 3;
Function<Integer, Integer> multiplyThenAdd = multiply.andThen(add);
Function<Integer, Integer> addThenMultiply = multiply.compose(add);

System.out.println(multiplyThenAdd.apply(5));  // (5 * 2) + 3 = 13
System.out.println(addThenMultiply.apply(5));  // (5 + 3) * 2 = 16

// Identity function
Function<String, String> identity = Function.identity();
String result = identity.apply("Hello");  // "Hello"
```

### 4.3 Predicate<T>

Represents a boolean-valued function of one argument.

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    
    // Default methods
    default Predicate<T> and(Predicate<? super T> other);
    default Predicate<T> or(Predicate<? super T> other);
    default Predicate<T> negate();
    static <T> Predicate<T> isEqual(Object targetRef);
}
```

**Examples:**

```java
// Basic usage
Predicate<String> isEmpty = s -> s.isEmpty();
boolean result = isEmpty.test("");  // true

// Combining predicates
Predicate<String> isNotEmpty = isEmpty.negate();
Predicate<String> hasLength = s -> s.length() > 5;
Predicate<String> isNotEmptyAndLong = isNotEmpty.and(hasLength);

List<String> words = Arrays.asList("", "hello", "world", "java", "programming");
List<String> filtered = words.stream()
    .filter(isNotEmptyAndLong)
    .collect(Collectors.toList());  // ["programming"]

// Using or
Predicate<String> startsWithA = s -> s.startsWith("A");
Predicate<String> startsWithB = s -> s.startsWith("B");
Predicate<String> startsWithAOrB = startsWithA.or(startsWithB);

// isEqual
Predicate<String> isHello = Predicate.isEqual("Hello");
boolean result = isHello.test("Hello");  // true
```

### 4.4 Consumer<T>

Represents an operation that accepts a single input argument and returns no result.

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    
    // Default method
    default Consumer<T> andThen(Consumer<? super T> after);
}
```

**Examples:**

```java
// Basic usage
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello");  // Prints "Hello"

// Chaining consumers
Consumer<String> print = System.out::print;
Consumer<String> printLine = System.out::println;
Consumer<String> printAndPrintLine = print.andThen(printLine);

printAndPrintLine.accept("Hello");  // Prints "Hello" then "Hello" with newline

// Using with collections
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(System.out::println);
```

### 4.5 Supplier<T>

Represents a supplier of results. No input, produces output.

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

**Examples:**

```java
// Basic usage
Supplier<String> stringSupplier = () -> "Hello";
String result = stringSupplier.get();  // "Hello"

// Lazy evaluation
Supplier<Double> randomValue = () -> Math.random();
double value1 = randomValue.get();
double value2 = randomValue.get();  // Different values

// Default value supplier
Supplier<String> defaultName = () -> "Unknown";
String name = Optional.ofNullable(nullName).orElseGet(defaultName);
```

### 4.6 BiFunction<T, U, R>

Represents a function that accepts two arguments and produces a result.

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
    
    // Default method
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after);
}
```

**Examples:**

```java
// Basic usage
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
Integer result = add.apply(5, 3);  // 8

// Chaining
BiFunction<Integer, Integer, Integer> multiply = (a, b) -> a * b;
Function<Integer, String> toString = String::valueOf;
BiFunction<Integer, Integer, String> multiplyAndToString = multiply.andThen(toString);

String result = multiplyAndToString.apply(5, 3);  // "15"
```

### 4.7 BinaryOperator<T>

Special case of BiFunction where both operands and result are of the same type.

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    // Static methods
    static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator);
    static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator);
}
```

**Examples:**

```java
// Basic usage
BinaryOperator<Integer> add = (a, b) -> a + b;
Integer result = add.apply(5, 3);  // 8

// minBy and maxBy
BinaryOperator<String> minLength = BinaryOperator.minBy(
    Comparator.comparing(String::length)
);
BinaryOperator<String> maxLength = BinaryOperator.maxBy(
    Comparator.comparing(String::length)
);

String min = minLength.apply("hello", "world");  // "hello"
String max = maxLength.apply("hello", "world");  // "world"
```

### 4.8 UnaryOperator<T>

Special case of Function where input and output are of the same type.

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    // Static method
    static <T> UnaryOperator<T> identity();
}
```

**Examples:**

```java
// Basic usage
UnaryOperator<String> toUpper = s -> s.toUpperCase();
String result = toUpper.apply("hello");  // "HELLO"

// Identity
UnaryOperator<String> identity = UnaryOperator.identity();
String result = identity.apply("hello");  // "hello"
```

### 4.9 Other Specialized Functional Interfaces

#### IntFunction, LongFunction, DoubleFunction

```java
IntFunction<String> intToString = i -> String.valueOf(i);
LongFunction<Double> longToDouble = l -> (double) l;
DoubleFunction<String> doubleToString = d -> String.valueOf(d);
```

#### ToIntFunction, ToLongFunction, ToDoubleFunction

```java
ToIntFunction<String> stringLength = String::length;
ToLongFunction<Integer> intToLong = i -> (long) i;
ToDoubleFunction<String> parseDouble = Double::parseDouble;
```

#### IntPredicate, LongPredicate, DoublePredicate

```java
IntPredicate isEven = i -> i % 2 == 0;
LongPredicate isPositive = l -> l > 0;
DoublePredicate isGreaterThanTen = d -> d > 10.0;
```

#### IntConsumer, LongConsumer, DoubleConsumer

```java
IntConsumer printInt = System.out::println;
LongConsumer printLong = System.out::println;
DoubleConsumer printDouble = System.out::println;
```

#### IntSupplier, LongSupplier, DoubleSupplier

```java
IntSupplier randomInt = () -> (int) (Math.random() * 100);
LongSupplier currentTime = System::currentTimeMillis;
DoubleSupplier randomDouble = Math::random;
```

---

## 5. Lambda Best Practices

### 5.1 Keep Lambdas Short and Focused

**Bad:**
```java
list.forEach(item -> {
    // 50 lines of complex logic
    if (condition1) {
        // ...
    }
    if (condition2) {
        // ...
    }
    // ...
});
```

**Good:**
```java
list.forEach(this::processItem);

private void processItem(Item item) {
    // Complex logic in a named method
}
```

### 5.2 Use Method References When Possible

**Bad:**
```java
list.stream()
    .map(s -> s.toUpperCase())
    .collect(Collectors.toList());
```

**Good:**
```java
list.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 5.3 Avoid Side Effects in Lambdas

**Bad:**
```java
List<String> result = new ArrayList<>();
list.forEach(s -> result.add(s.toUpperCase()));  // Side effect
```

**Good:**
```java
List<String> result = list.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 5.4 Use Descriptive Variable Names

**Bad:**
```java
list.stream()
    .filter(x -> x.length() > 5)
    .map(y -> y.toUpperCase())
    .collect(Collectors.toList());
```

**Good:**
```java
list.stream()
    .filter(word -> word.length() > 5)
    .map(word -> word.toUpperCase())
    .collect(Collectors.toList());
```

### 5.5 Prefer Streams Over Loops for Collections

**Bad:**
```java
List<String> result = new ArrayList<>();
for (String s : list) {
    if (s.length() > 5) {
        result.add(s.toUpperCase());
    }
}
```

**Good:**
```java
List<String> result = list.stream()
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 5.6 Handle Exceptions Properly

**Bad:**
```java
list.stream()
    .map(s -> Integer.parseInt(s))  // Can throw NumberFormatException
    .collect(Collectors.toList());
```

**Good:**
```java
// Option 1: Handle in lambda
list.stream()
    .map(s -> {
        try {
            return Integer.parseInt(s);
        } catch (NumberFormatException e) {
            return 0;  // or handle appropriately
        }
    })
    .collect(Collectors.toList());

// Option 2: Extract to method
list.stream()
    .map(this::parseIntSafely)
    .collect(Collectors.toList());

private int parseIntSafely(String s) {
    try {
        return Integer.parseInt(s);
    } catch (NumberFormatException e) {
        return 0;
    }
}
```

### 5.7 Use Parallel Streams Wisely

**Considerations:**
- Use for large datasets
- Operations must be stateless
- Avoid shared mutable state
- Consider overhead

```java
// Good: Large dataset, stateless operations
List<Integer> largeList = // ... large list
List<Integer> result = largeList.parallelStream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * 2)
    .collect(Collectors.toList());

// Bad: Small dataset (overhead > benefit)
List<Integer> smallList = Arrays.asList(1, 2, 3, 4, 5);
List<Integer> result = smallList.parallelStream()  // Unnecessary
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

---

## Summary of Part 1

**Key Takeaways:**
1. **Lambda expressions** provide concise syntax for functional interfaces
2. **Functional interfaces** have exactly one abstract method
3. **Method references** are shorthand for lambda expressions calling existing methods
4. **Built-in functional interfaces** in `java.util.function` cover common use cases
5. **Best practices** include keeping lambdas short, using method references, and avoiding side effects

**Next**: Part 2 will cover Stream API fundamentals, intermediate operations, and terminal operations.

