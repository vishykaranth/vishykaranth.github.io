# Java Design Patterns: Complete Guide with Examples

## Part 8: Behavioral Patterns - Visitor, Interpreter, Null Object

---

## Table of Contents

1. [Visitor Pattern](#1-visitor-pattern)
2. [Interpreter Pattern](#2-interpreter-pattern)
3. [Null Object Pattern](#3-null-object-pattern)

---

## 1. Visitor Pattern

### 1.1 Intent

Represent an operation to be performed on elements of an object structure. Lets you define a new operation without changing the classes of the elements on which it operates.

### 1.2 When to Use

- Need to perform operations on object structure
- Operations vary by element type
- Don't want to pollute element classes with operations
- Operations change frequently
- Object structure is stable

### 1.3 Scenarios

- **Compiler**: Different operations on AST nodes (type checking, code generation)
- **Document Processing**: Different operations on document elements (export, print)
- **File System**: Different operations on files/directories (search, backup)
- **Shopping Cart**: Different operations on items (calculate tax, apply discount)

### 1.4 Implementation

```java
// Element interface
interface Element {
    void accept(Visitor visitor);
}

// Concrete elements
class ConcreteElementA implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visitElementA(this);
    }
    
    public String operationA() {
        return "Element A specific operation";
    }
}

class ConcreteElementB implements Element {
    @Override
    public void accept(Visitor visitor) {
        visitor.visitElementB(this);
    }
    
    public String operationB() {
        return "Element B specific operation";
    }
}

// Visitor interface
interface Visitor {
    void visitElementA(ConcreteElementA element);
    void visitElementB(ConcreteElementB element);
}

// Concrete visitors
class ConcreteVisitor1 implements Visitor {
    @Override
    public void visitElementA(ConcreteElementA element) {
        System.out.println("Visitor 1: " + element.operationA());
    }
    
    @Override
    public void visitElementB(ConcreteElementB element) {
        System.out.println("Visitor 1: " + element.operationB());
    }
}

class ConcreteVisitor2 implements Visitor {
    @Override
    public void visitElementA(ConcreteElementA element) {
        System.out.println("Visitor 2: Processing " + element.operationA());
    }
    
    @Override
    public void visitElementB(ConcreteElementB element) {
        System.out.println("Visitor 2: Processing " + element.operationB());
    }
}
```

### 1.5 Usage Example

```java
import java.util.*;

public class VisitorExample {
    public static void main(String[] args) {
        List<Element> elements = Arrays.asList(
            new ConcreteElementA(),
            new ConcreteElementB()
        );
        
        Visitor visitor1 = new ConcreteVisitor1();
        for (Element element : elements) {
            element.accept(visitor1);
        }
        
        System.out.println();
        
        Visitor visitor2 = new ConcreteVisitor2();
        for (Element element : elements) {
            element.accept(visitor2);
        }
    }
}
```

### 1.6 Real-World Example: Shopping Cart

```java
import java.util.*;

// Element interface
interface ShoppingItem {
    double accept(ShoppingCartVisitor visitor);
}

// Concrete elements
class Book implements ShoppingItem {
    private double price;
    private String isbn;
    
    public Book(double price, String isbn) {
        this.price = price;
        this.isbn = isbn;
    }
    
    public double getPrice() { return price; }
    public String getIsbn() { return isbn; }
    
    @Override
    public double accept(ShoppingCartVisitor visitor) {
        return visitor.visit(this);
    }
}

class Fruit implements ShoppingItem {
    private double pricePerKg;
    private double weight;
    private String name;
    
    public Fruit(double pricePerKg, double weight, String name) {
        this.pricePerKg = pricePerKg;
        this.weight = weight;
        this.name = name;
    }
    
    public double getPricePerKg() { return pricePerKg; }
    public double getWeight() { return weight; }
    public String getName() { return name; }
    
    @Override
    public double accept(ShoppingCartVisitor visitor) {
        return visitor.visit(this);
    }
}

// Visitor interface
interface ShoppingCartVisitor {
    double visit(Book book);
    double visit(Fruit fruit);
}

// Concrete visitors
class ShoppingCartVisitorImpl implements ShoppingCartVisitor {
    @Override
    public double visit(Book book) {
        double cost = book.getPrice();
        // Apply 5% discount on books
        if (book.getPrice() > 50) {
            cost = book.getPrice() - 5;
        }
        System.out.println("Book ISBN: " + book.getIsbn() + " cost: $" + cost);
        return cost;
    }
    
    @Override
    public double visit(Fruit fruit) {
        double cost = fruit.getPricePerKg() * fruit.getWeight();
        System.out.println(fruit.getName() + " cost: $" + cost);
        return cost;
    }
}

// Usage
public class ShoppingCartVisitorExample {
    public static void main(String[] args) {
        List<ShoppingItem> items = Arrays.asList(
            new Book(20, "1234"),
            new Book(100, "5678"),
            new Fruit(10, 2, "Banana"),
            new Fruit(5, 5, "Apple")
        );
        
        ShoppingCartVisitor visitor = new ShoppingCartVisitorImpl();
        double total = 0;
        
        for (ShoppingItem item : items) {
            total += item.accept(visitor);
        }
        
        System.out.println("Total cost: $" + total);
    }
}
```

---

## 2. Interpreter Pattern

### 2.1 Intent

Given a language, define a representation for its grammar along with an interpreter that uses the representation to interpret sentences in the language.

### 2.2 When to Use

- Have a language to interpret
- Grammar is simple
- Performance is not critical
- Want to represent grammar as class hierarchy

### 2.3 Scenarios

- **Expression Evaluation**: Mathematical expressions, SQL queries
- **Rule Engines**: Business rules, validation rules
- **Query Languages**: Database queries, search queries
- **Configuration Languages**: DSL parsing
- **Regular Expressions**: Pattern matching

### 2.4 Implementation

```java
// Abstract expression
interface Expression {
    boolean interpret(String context);
}

// Terminal expressions
class TerminalExpression implements Expression {
    private String data;
    
    public TerminalExpression(String data) {
        this.data = data;
    }
    
    @Override
    public boolean interpret(String context) {
        return context.contains(data);
    }
}

// Non-terminal expressions
class OrExpression implements Expression {
    private Expression expr1;
    private Expression expr2;
    
    public OrExpression(Expression expr1, Expression expr2) {
        this.expr1 = expr1;
        this.expr2 = expr2;
    }
    
    @Override
    public boolean interpret(String context) {
        return expr1.interpret(context) || expr2.interpret(context);
    }
}

class AndExpression implements Expression {
    private Expression expr1;
    private Expression expr2;
    
    public AndExpression(Expression expr1, Expression expr2) {
        this.expr1 = expr1;
        this.expr2 = expr2;
    }
    
    @Override
    public boolean interpret(String context) {
        return expr1.interpret(context) && expr2.interpret(context);
    }
}
```

### 2.5 Usage Example

```java
public class InterpreterExample {
    public static void main(String[] args) {
        Expression robert = new TerminalExpression("Robert");
        Expression john = new TerminalExpression("John");
        Expression isMale = new OrExpression(robert, john);
        
        Expression julie = new TerminalExpression("Julie");
        Expression married = new TerminalExpression("Married");
        Expression isMarriedWoman = new AndExpression(julie, married);
        
        System.out.println("John is male? " + isMale.interpret("John"));
        System.out.println("Julie is a married woman? " + isMarriedWoman.interpret("Married Julie"));
    }
}
```

### 2.6 Real-World Example: Expression Evaluator

```java
// Expression interface
interface MathExpression {
    int interpret();
}

// Terminal expressions
class Number implements MathExpression {
    private int value;
    
    public Number(int value) {
        this.value = value;
    }
    
    @Override
    public int interpret() {
        return value;
    }
}

// Non-terminal expressions
class Add implements MathExpression {
    private MathExpression left;
    private MathExpression right;
    
    public Add(MathExpression left, MathExpression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() + right.interpret();
    }
}

class Subtract implements MathExpression {
    private MathExpression left;
    private MathExpression right;
    
    public Subtract(MathExpression left, MathExpression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() - right.interpret();
    }
}

class Multiply implements MathExpression {
    private MathExpression left;
    private MathExpression right;
    
    public Multiply(MathExpression left, MathExpression right) {
        this.left = left;
        this.right = right;
    }
    
    @Override
    public int interpret() {
        return left.interpret() * right.interpret();
    }
}

// Usage
public class ExpressionEvaluatorExample {
    public static void main(String[] args) {
        // Expression: (10 + 5) * 2
        MathExpression expression = new Multiply(
            new Add(new Number(10), new Number(5)),
            new Number(2)
        );
        
        System.out.println("Result: " + expression.interpret());
        
        // Expression: 20 - 5
        MathExpression expression2 = new Subtract(
            new Number(20),
            new Number(5)
        );
        
        System.out.println("Result: " + expression2.interpret());
    }
}
```

---

## 3. Null Object Pattern

### 3.1 Intent

Provide an object as a surrogate for the absence of an object. A null object provides default behavior when nothing is available.

### 3.2 When to Use

- Want to avoid null checks
- Need default behavior for null cases
- Want to eliminate NullPointerException
- Need to provide meaningful default behavior

### 3.3 Scenarios

- **Logging**: Null logger that does nothing
- **Caching**: Null cache that always misses
- **Event Handlers**: Null handler that does nothing
- **Optional Dependencies**: Default implementations
- **Collections**: Empty collections instead of null

### 3.4 Implementation

```java
// Abstract class
abstract class AbstractCustomer {
    protected String name;
    
    public abstract boolean isNil();
    public abstract String getName();
}

// Real customer
class RealCustomer extends AbstractCustomer {
    public RealCustomer(String name) {
        this.name = name;
    }
    
    @Override
    public boolean isNil() {
        return false;
    }
    
    @Override
    public String getName() {
        return name;
    }
}

// Null customer
class NullCustomer extends AbstractCustomer {
    @Override
    public boolean isNil() {
        return true;
    }
    
    @Override
    public String getName() {
        return "Not Available";
    }
}

// Customer factory
class CustomerFactory {
    private static final String[] names = {"Rob", "Joe", "Julie"};
    
    public static AbstractCustomer getCustomer(String name) {
        for (String n : names) {
            if (n.equalsIgnoreCase(name)) {
                return new RealCustomer(name);
            }
        }
        return new NullCustomer();
    }
}
```

### 3.5 Usage Example

```java
public class NullObjectExample {
    public static void main(String[] args) {
        AbstractCustomer customer1 = CustomerFactory.getCustomer("Rob");
        AbstractCustomer customer2 = CustomerFactory.getCustomer("Bob");
        AbstractCustomer customer3 = CustomerFactory.getCustomer("Julie");
        AbstractCustomer customer4 = CustomerFactory.getCustomer("Laura");
        
        System.out.println("Customers:");
        System.out.println(customer1.getName());
        System.out.println(customer2.getName());
        System.out.println(customer3.getName());
        System.out.println(customer4.getName());
    }
}
```

### 3.6 Real-World Example: Logger

```java
// Logger interface
interface Logger {
    void log(String message);
    void error(String message);
    void warn(String message);
}

// Real logger
class ConsoleLogger implements Logger {
    @Override
    public void log(String message) {
        System.out.println("[LOG] " + message);
    }
    
    @Override
    public void error(String message) {
        System.err.println("[ERROR] " + message);
    }
    
    @Override
    public void warn(String message) {
        System.out.println("[WARN] " + message);
    }
}

// Null logger
class NullLogger implements Logger {
    @Override
    public void log(String message) {
        // Do nothing
    }
    
    @Override
    public void error(String message) {
        // Do nothing
    }
    
    @Override
    public void warn(String message) {
        // Do nothing
    }
}

// Service using logger
class UserService {
    private Logger logger;
    
    public UserService(Logger logger) {
        this.logger = logger != null ? logger : new NullLogger();
    }
    
    public void createUser(String username) {
        logger.log("Creating user: " + username);
        // Create user logic
        logger.log("User created: " + username);
    }
    
    public void deleteUser(String username) {
        logger.warn("Deleting user: " + username);
        // Delete user logic
        logger.log("User deleted: " + username);
    }
}

// Usage
public class NullLoggerExample {
    public static void main(String[] args) {
        // With real logger
        UserService service1 = new UserService(new ConsoleLogger());
        service1.createUser("john");
        
        System.out.println();
        
        // With null logger (no logging)
        UserService service2 = new UserService(null);
        service2.createUser("jane");
    }
}
```

### 3.7 Collection Null Object

```java
import java.util.*;

// Null collection
class NullList<T> implements List<T> {
    @Override
    public int size() { return 0; }
    
    @Override
    public boolean isEmpty() { return true; }
    
    @Override
    public boolean contains(Object o) { return false; }
    
    @Override
    public Iterator<T> iterator() {
        return Collections.emptyIterator();
    }
    
    @Override
    public Object[] toArray() { return new Object[0]; }
    
    @Override
    public <T1> T1[] toArray(T1[] a) { return a; }
    
    @Override
    public boolean add(T t) { return false; }
    
    @Override
    public boolean remove(Object o) { return false; }
    
    @Override
    public boolean containsAll(Collection<?> c) { return false; }
    
    @Override
    public boolean addAll(Collection<? extends T> c) { return false; }
    
    @Override
    public boolean addAll(int index, Collection<? extends T> c) { return false; }
    
    @Override
    public boolean removeAll(Collection<?> c) { return false; }
    
    @Override
    public boolean retainAll(Collection<?> c) { return false; }
    
    @Override
    public void clear() {}
    
    @Override
    public T get(int index) { return null; }
    
    @Override
    public T set(int index, T element) { return null; }
    
    @Override
    public void add(int index, T element) {}
    
    @Override
    public T remove(int index) { return null; }
    
    @Override
    public int indexOf(Object o) { return -1; }
    
    @Override
    public int lastIndexOf(Object o) { return -1; }
    
    @Override
    public ListIterator<T> listIterator() {
        return Collections.emptyListIterator();
    }
    
    @Override
    public ListIterator<T> listIterator(int index) {
        return Collections.emptyListIterator();
    }
    
    @Override
    public List<T> subList(int fromIndex, int toIndex) {
        return new NullList<>();
    }
}

// Usage
public class NullCollectionExample {
    public static void main(String[] args) {
        List<String> list = new NullList<>();
        
        // No null pointer exception
        System.out.println("Size: " + list.size());
        System.out.println("Is empty: " + list.isEmpty());
        
        // Safe iteration
        for (String item : list) {
            System.out.println(item); // Never executes
        }
    }
}
```

---

## Summary: Part 8

### Patterns Covered

1. **Visitor**: Add operations to object structure
2. **Interpreter**: Interpret language expressions
3. **Null Object**: Provide default behavior for null

### Key Takeaways

- **Visitor**: Use when operations vary by element type
- **Interpreter**: Use for simple language interpretation
- **Null Object**: Use to avoid null checks and provide defaults

### Next Steps

**Part 9** will cover:
- Additional Patterns (Dependency Injection, MVC)
- Concurrency Patterns
- Modern Patterns

---

**Master these patterns for advanced software design!**

