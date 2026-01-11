# Java Design Patterns: Complete Guide with Examples

## Part 6: Behavioral Patterns - Chain of Responsibility, Iterator, Mediator

---

## Table of Contents

1. [Chain of Responsibility Pattern](#1-chain-of-responsibility-pattern)
2. [Iterator Pattern](#2-iterator-pattern)
3. [Mediator Pattern](#3-mediator-pattern)

---

## 1. Chain of Responsibility Pattern

### 1.1 Intent

Avoid coupling the sender of a request to its receiver by giving more than one object a chance to handle the request. Chain the receiving objects and pass the request along the chain until an object handles it.

### 1.2 When to Use

- More than one object may handle a request
- Don't know which handler should process the request
- Want to specify handlers dynamically
- Want to decouple request sender and receiver

### 1.3 Scenarios

- **Request Processing**: HTTP request filters, middleware
- **Logging**: Different log levels (DEBUG, INFO, WARN, ERROR)
- **Approval Workflow**: Multi-level approval system
- **Exception Handling**: Try-catch chain
- **Event Handling**: Event propagation

### 1.4 Implementation

```java
// Handler interface
abstract class Logger {
    public static int INFO = 1;
    public static int DEBUG = 2;
    public static int ERROR = 3;
    
    protected int level;
    protected Logger nextLogger;
    
    public void setNextLogger(Logger nextLogger) {
        this.nextLogger = nextLogger;
    }
    
    public void logMessage(int level, String message) {
        if (this.level <= level) {
            write(message);
        }
        if (nextLogger != null) {
            nextLogger.logMessage(level, message);
        }
    }
    
    abstract protected void write(String message);
}

// Concrete handlers
class ConsoleLogger extends Logger {
    public ConsoleLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("Console Logger: " + message);
    }
}

class FileLogger extends Logger {
    public FileLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("File Logger: " + message);
    }
}

class ErrorLogger extends Logger {
    public ErrorLogger(int level) {
        this.level = level;
    }
    
    @Override
    protected void write(String message) {
        System.out.println("Error Logger: " + message);
    }
}
```

### 1.5 Usage Example

```java
public class ChainOfResponsibilityExample {
    private static Logger getChainOfLoggers() {
        Logger errorLogger = new ErrorLogger(Logger.ERROR);
        Logger fileLogger = new FileLogger(Logger.DEBUG);
        Logger consoleLogger = new ConsoleLogger(Logger.INFO);
        
        errorLogger.setNextLogger(fileLogger);
        fileLogger.setNextLogger(consoleLogger);
        
        return errorLogger;
    }
    
    public static void main(String[] args) {
        Logger loggerChain = getChainOfLoggers();
        
        loggerChain.logMessage(Logger.INFO, "This is an information.");
        System.out.println();
        loggerChain.logMessage(Logger.DEBUG, "This is a debug level information.");
        System.out.println();
        loggerChain.logMessage(Logger.ERROR, "This is an error information.");
    }
}
```

### 1.6 Real-World Example: Approval Workflow

```java
// Handler interface
abstract class Approver {
    protected Approver nextApprover;
    protected String name;
    protected double approvalLimit;
    
    public Approver(String name, double approvalLimit) {
        this.name = name;
        this.approvalLimit = approvalLimit;
    }
    
    public void setNextApprover(Approver nextApprover) {
        this.nextApprover = nextApprover;
    }
    
    public void processRequest(PurchaseRequest request) {
        if (request.getAmount() <= approvalLimit) {
            approve(request);
        } else if (nextApprover != null) {
            nextApprover.processRequest(request);
        } else {
            reject(request);
        }
    }
    
    protected void approve(PurchaseRequest request) {
        System.out.println(name + " approved purchase request #" + 
                          request.getRequestNumber() + " for $" + request.getAmount());
    }
    
    protected void reject(PurchaseRequest request) {
        System.out.println("Purchase request #" + request.getRequestNumber() + 
                          " for $" + request.getAmount() + " cannot be approved");
    }
}

// Request class
class PurchaseRequest {
    private int requestNumber;
    private double amount;
    private String purpose;
    
    public PurchaseRequest(int requestNumber, double amount, String purpose) {
        this.requestNumber = requestNumber;
        this.amount = amount;
        this.purpose = purpose;
    }
    
    public int getRequestNumber() { return requestNumber; }
    public double getAmount() { return amount; }
    public String getPurpose() { return purpose; }
}

// Concrete handlers
class Manager extends Approver {
    public Manager(String name) {
        super(name, 1000);
    }
}

class Director extends Approver {
    public Director(String name) {
        super(name, 5000);
    }
}

class VP extends Approver {
    public VP(String name) {
        super(name, 10000);
    }
}

class President extends Approver {
    public President(String name) {
        super(name, Double.MAX_VALUE);
    }
}

// Usage
public class ApprovalWorkflowExample {
    public static void main(String[] args) {
        Approver manager = new Manager("John");
        Approver director = new Director("Jane");
        Approver vp = new VP("Bob");
        Approver president = new President("Alice");
        
        // Set up chain
        manager.setNextApprover(director);
        director.setNextApprover(vp);
        vp.setNextApprover(president);
        
        // Process requests
        manager.processRequest(new PurchaseRequest(1, 500, "Office supplies"));
        System.out.println();
        manager.processRequest(new PurchaseRequest(2, 2500, "Equipment"));
        System.out.println();
        manager.processRequest(new PurchaseRequest(3, 7500, "Software license"));
        System.out.println();
        manager.processRequest(new PurchaseRequest(4, 15000, "Infrastructure"));
    }
}
```

---

## 2. Iterator Pattern

### 2.1 Intent

Provide a way to access the elements of an aggregate object sequentially without exposing its underlying representation.

### 2.2 When to Use

- Need to traverse different data structures uniformly
- Want to hide implementation details of collection
- Need multiple ways to traverse same collection
- Want to provide a standard traversal interface

### 2.3 Scenarios

- **Collections**: Iterate over lists, sets, maps
- **Tree Traversal**: Different traversal orders (preorder, inorder, postorder)
- **Database Results**: Iterate over query results
- **File System**: Iterate over files and directories

### 2.4 Implementation

```java
// Iterator interface
interface Iterator<T> {
    boolean hasNext();
    T next();
    void remove();
}

// Aggregate interface
interface Container<T> {
    Iterator<T> getIterator();
}

// Concrete aggregate
class NameRepository implements Container<String> {
    private String[] names = {"Robert", "John", "Julie", "Lora"};
    
    @Override
    public Iterator<String> getIterator() {
        return new NameIterator();
    }
    
    // Inner iterator class
    private class NameIterator implements Iterator<String> {
        int index;
        
        @Override
        public boolean hasNext() {
            return index < names.length;
        }
        
        @Override
        public String next() {
            if (hasNext()) {
                return names[index++];
            }
            return null;
        }
        
        @Override
        public void remove() {
            throw new UnsupportedOperationException();
        }
    }
}
```

### 2.5 Usage Example

```java
public class IteratorExample {
    public static void main(String[] args) {
        NameRepository namesRepository = new NameRepository();
        
        Iterator<String> iterator = namesRepository.getIterator();
        while (iterator.hasNext()) {
            String name = iterator.next();
            System.out.println("Name: " + name);
        }
    }
}
```

### 2.6 Real-World Example: Custom Collection Iterator

```java
import java.util.*;

// Custom collection
class BookCollection {
    private List<Book> books;
    
    public BookCollection() {
        this.books = new ArrayList<>();
    }
    
    public void addBook(Book book) {
        books.add(book);
    }
    
    public Iterator<Book> getIterator() {
        return new BookIterator();
    }
    
    private class BookIterator implements Iterator<Book> {
        private int index = 0;
        
        @Override
        public boolean hasNext() {
            return index < books.size();
        }
        
        @Override
        public Book next() {
            if (hasNext()) {
                return books.get(index++);
            }
            throw new NoSuchElementException();
        }
        
        @Override
        public void remove() {
            if (index > 0) {
                books.remove(--index);
            } else {
                throw new IllegalStateException();
            }
        }
    }
}

class Book {
    private String title;
    private String author;
    
    public Book(String title, String author) {
        this.title = title;
        this.author = author;
    }
    
    public String getTitle() { return title; }
    public String getAuthor() { return author; }
    
    @Override
    public String toString() {
        return title + " by " + author;
    }
}

// Usage
public class BookCollectionExample {
    public static void main(String[] args) {
        BookCollection collection = new BookCollection();
        collection.addBook(new Book("Design Patterns", "Gang of Four"));
        collection.addBook(new Book("Clean Code", "Robert Martin"));
        collection.addBook(new Book("Refactoring", "Martin Fowler"));
        
        Iterator<Book> iterator = collection.getIterator();
        while (iterator.hasNext()) {
            System.out.println(iterator.next());
        }
    }
}
```

### 2.7 Java Built-in Iterator

```java
import java.util.*;

public class JavaIteratorExample {
    public static void main(String[] args) {
        // List iterator
        List<String> list = Arrays.asList("Apple", "Banana", "Cherry");
        Iterator<String> listIterator = list.iterator();
        while (listIterator.hasNext()) {
            System.out.println(listIterator.next());
        }
        
        System.out.println();
        
        // Set iterator
        Set<Integer> set = new HashSet<>(Arrays.asList(1, 2, 3, 4, 5));
        Iterator<Integer> setIterator = set.iterator();
        while (setIterator.hasNext()) {
            System.out.println(setIterator.next());
        }
        
        System.out.println();
        
        // Map iterator (via entrySet)
        Map<String, Integer> map = new HashMap<>();
        map.put("One", 1);
        map.put("Two", 2);
        map.put("Three", 3);
        
        Iterator<Map.Entry<String, Integer>> mapIterator = map.entrySet().iterator();
        while (mapIterator.hasNext()) {
            Map.Entry<String, Integer> entry = mapIterator.next();
            System.out.println(entry.getKey() + " = " + entry.getValue());
        }
    }
}
```

---

## 3. Mediator Pattern

### 3.1 Intent

Define an object that encapsulates how a set of objects interact. Mediator promotes loose coupling by keeping objects from referring to each other explicitly.

### 3.2 When to Use

- Set of objects communicate in complex ways
- Reusing an object is difficult because it refers to many other objects
- Want to customize behavior distributed between classes
- Objects have too many direct dependencies

### 3.3 Scenarios

- **Chat Application**: Users communicate through chat room (mediator)
- **Air Traffic Control**: Planes communicate through control tower
- **GUI Components**: Components communicate through dialog/window
- **Event Bus**: Components communicate through event bus
- **Workflow Management**: Tasks communicate through workflow engine

### 3.4 Implementation

```java
import java.util.*;

// Mediator interface
interface ChatMediator {
    void sendMessage(String message, User user);
    void addUser(User user);
}

// Concrete mediator
class ChatRoom implements ChatMediator {
    private List<User> users;
    
    public ChatRoom() {
        this.users = new ArrayList<>();
    }
    
    @Override
    public void addUser(User user) {
        users.add(user);
    }
    
    @Override
    public void sendMessage(String message, User user) {
        for (User u : users) {
            if (u != user) {
                u.receive(message);
            }
        }
    }
}

// Colleague
abstract class User {
    protected ChatMediator mediator;
    protected String name;
    
    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
    }
    
    public abstract void send(String message);
    public abstract void receive(String message);
}

// Concrete colleague
class ChatUser extends User {
    public ChatUser(ChatMediator mediator, String name) {
        super(mediator, name);
    }
    
    @Override
    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this);
    }
    
    @Override
    public void receive(String message) {
        System.out.println(name + " receives: " + message);
    }
}
```

### 3.5 Usage Example

```java
public class MediatorExample {
    public static void main(String[] args) {
        ChatMediator chatRoom = new ChatRoom();
        
        User user1 = new ChatUser(chatRoom, "Alice");
        User user2 = new ChatUser(chatRoom, "Bob");
        User user3 = new ChatUser(chatRoom, "Charlie");
        
        chatRoom.addUser(user1);
        chatRoom.addUser(user2);
        chatRoom.addUser(user3);
        
        user1.send("Hello everyone!");
        System.out.println();
        user2.send("Hi Alice!");
    }
}
```

### 3.6 Real-World Example: Air Traffic Control

```java
import java.util.*;

// Mediator
interface AirTrafficControl {
    void requestLanding(Aircraft aircraft);
    void notifyTakeoff(Aircraft aircraft);
    void registerAircraft(Aircraft aircraft);
}

// Concrete mediator
class ControlTower implements AirTrafficControl {
    private List<Aircraft> aircrafts;
    private Queue<Aircraft> landingQueue;
    
    public ControlTower() {
        this.aircrafts = new ArrayList<>();
        this.landingQueue = new LinkedList<>();
    }
    
    @Override
    public void registerAircraft(Aircraft aircraft) {
        aircrafts.add(aircraft);
    }
    
    @Override
    public void requestLanding(Aircraft aircraft) {
        System.out.println("Control Tower: " + aircraft.getName() + " requesting landing");
        landingQueue.offer(aircraft);
        processLanding();
    }
    
    @Override
    public void notifyTakeoff(Aircraft aircraft) {
        System.out.println("Control Tower: " + aircraft.getName() + " has taken off");
        processLanding();
    }
    
    private void processLanding() {
        if (!landingQueue.isEmpty()) {
            Aircraft aircraft = landingQueue.poll();
            System.out.println("Control Tower: " + aircraft.getName() + " cleared for landing");
            aircraft.land();
        }
    }
}

// Colleague
abstract class Aircraft {
    protected AirTrafficControl atc;
    protected String name;
    
    public Aircraft(AirTrafficControl atc, String name) {
        this.atc = atc;
        this.name = name;
        atc.registerAircraft(this);
    }
    
    public String getName() {
        return name;
    }
    
    public abstract void requestLanding();
    public abstract void land();
    public abstract void takeoff();
}

// Concrete colleague
class Airplane extends Aircraft {
    public Airplane(AirTrafficControl atc, String name) {
        super(atc, name);
    }
    
    @Override
    public void requestLanding() {
        atc.requestLanding(this);
    }
    
    @Override
    public void land() {
        System.out.println(name + " is landing");
    }
    
    @Override
    public void takeoff() {
        System.out.println(name + " is taking off");
        atc.notifyTakeoff(this);
    }
}

// Usage
public class AirTrafficControlExample {
    public static void main(String[] args) {
        AirTrafficControl tower = new ControlTower();
        
        Aircraft plane1 = new Airplane(tower, "Flight 101");
        Aircraft plane2 = new Airplane(tower, "Flight 202");
        Aircraft plane3 = new Airplane(tower, "Flight 303");
        
        plane1.requestLanding();
        System.out.println();
        plane2.requestLanding();
        System.out.println();
        plane1.takeoff();
        System.out.println();
        plane3.requestLanding();
    }
}
```

### 3.7 GUI Components Example

```java
// Mediator
interface DialogMediator {
    void notify(Component component, String event);
}

// Concrete mediator
class Dialog implements DialogMediator {
    private Button button;
    private TextField textField;
    private Checkbox checkbox;
    
    public void setComponents(Button button, TextField textField, Checkbox checkbox) {
        this.button = button;
        this.textField = textField;
        this.checkbox = checkbox;
    }
    
    @Override
    public void notify(Component component, String event) {
        if (component == button && "clicked".equals(event)) {
            if (checkbox.isChecked()) {
                System.out.println("Button clicked: " + textField.getText());
            } else {
                System.out.println("Checkbox must be checked");
            }
        } else if (component == checkbox && "checked".equals(event)) {
            button.setEnabled(true);
        } else if (component == checkbox && "unchecked".equals(event)) {
            button.setEnabled(false);
        }
    }
}

// Colleague
abstract class Component {
    protected DialogMediator mediator;
    
    public Component(DialogMediator mediator) {
        this.mediator = mediator;
    }
    
    public void changed() {
        mediator.notify(this, "changed");
    }
}

// Concrete colleagues
class Button extends Component {
    private boolean enabled = false;
    private String label;
    
    public Button(DialogMediator mediator, String label) {
        super(mediator);
        this.label = label;
    }
    
    public void click() {
        if (enabled) {
            mediator.notify(this, "clicked");
        }
    }
    
    public void setEnabled(boolean enabled) {
        this.enabled = enabled;
        System.out.println("Button " + label + " enabled: " + enabled);
    }
}

class TextField extends Component {
    private String text = "";
    
    public TextField(DialogMediator mediator) {
        super(mediator);
    }
    
    public void setText(String text) {
        this.text = text;
        changed();
    }
    
    public String getText() {
        return text;
    }
}

class Checkbox extends Component {
    private boolean checked = false;
    
    public Checkbox(DialogMediator mediator) {
        super(mediator);
    }
    
    public void check() {
        checked = true;
        mediator.notify(this, "checked");
    }
    
    public void uncheck() {
        checked = false;
        mediator.notify(this, "unchecked");
    }
    
    public boolean isChecked() {
        return checked;
    }
}

// Usage
public class GUIComponentsExample {
    public static void main(String[] args) {
        Dialog dialog = new Dialog();
        
        Button button = new Button(dialog, "Submit");
        TextField textField = new TextField(dialog);
        Checkbox checkbox = new Checkbox(dialog);
        
        dialog.setComponents(button, textField, checkbox);
        
        textField.setText("Hello World");
        checkbox.check();
        button.click();
    }
}
```

---

## Summary: Part 6

### Patterns Covered

1. **Chain of Responsibility**: Pass requests along a chain of handlers
2. **Iterator**: Traverse collections without exposing implementation
3. **Mediator**: Centralize communication between objects

### Key Takeaways

- **Chain of Responsibility**: Use for request processing pipelines
- **Iterator**: Use for uniform collection traversal
- **Mediator**: Use to reduce coupling between communicating objects

### Next Steps

**Part 7** will cover:
- Memento Pattern
- State Pattern
- Template Method Pattern

---

**Master these behavioral patterns for flexible object interactions!**

