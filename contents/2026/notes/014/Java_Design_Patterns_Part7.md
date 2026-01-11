# Java Design Patterns: Complete Guide with Examples

## Part 7: Behavioral Patterns - Memento, State, Template Method

---

## Table of Contents

1. [Memento Pattern](#1-memento-pattern)
2. [State Pattern](#2-state-pattern)
3. [Template Method Pattern](#3-template-method-pattern)

---

## 1. Memento Pattern

### 1.1 Intent

Capture and externalize an object's internal state so that the object can be restored to this state later, without violating encapsulation.

### 1.2 When to Use

- Need to save and restore object state
- Direct access to object's state would violate encapsulation
- Need to implement undo/redo functionality
- Need to implement checkpoints/snapshots

### 1.3 Scenarios

- **Text Editor**: Undo/redo operations
- **Game State**: Save/load game progress
- **Database Transactions**: Rollback to previous state
- **Configuration Management**: Restore previous configurations
- **Version Control**: Revert to previous versions

### 1.4 Implementation

```java
// Memento
class Memento {
    private String state;
    
    public Memento(String state) {
        this.state = state;
    }
    
    public String getState() {
        return state;
    }
}

// Originator
class TextEditor {
    private String content;
    
    public void setContent(String content) {
        this.content = content;
        System.out.println("Content set to: " + content);
    }
    
    public String getContent() {
        return content;
    }
    
    public Memento save() {
        return new Memento(content);
    }
    
    public void restore(Memento memento) {
        content = memento.getState();
        System.out.println("Content restored to: " + content);
    }
}

// Caretaker
class History {
    private java.util.Stack<Memento> history;
    
    public History() {
        this.history = new java.util.Stack<>();
    }
    
    public void save(Memento memento) {
        history.push(memento);
    }
    
    public Memento undo() {
        if (!history.isEmpty()) {
            history.pop(); // Remove current state
            if (!history.isEmpty()) {
                return history.peek(); // Return previous state
            }
        }
        return null;
    }
}
```

### 1.5 Usage Example

```java
public class MementoExample {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        History history = new History();
        
        editor.setContent("Version 1");
        history.save(editor.save());
        
        editor.setContent("Version 2");
        history.save(editor.save());
        
        editor.setContent("Version 3");
        history.save(editor.save());
        
        System.out.println("\nCurrent content: " + editor.getContent());
        
        System.out.println("Undoing...");
        Memento previous = history.undo();
        if (previous != null) {
            editor.restore(previous);
        }
    }
}
```

### 1.6 Real-World Example: Game State

```java
import java.util.*;

// Memento
class GameMemento {
    private int level;
    private int score;
    private String playerName;
    
    public GameMemento(int level, int score, String playerName) {
        this.level = level;
        this.score = score;
        this.playerName = playerName;
    }
    
    public int getLevel() { return level; }
    public int getScore() { return score; }
    public String getPlayerName() { return playerName; }
}

// Originator
class Game {
    private int level;
    private int score;
    private String playerName;
    
    public Game(String playerName) {
        this.playerName = playerName;
        this.level = 1;
        this.score = 0;
    }
    
    public void play() {
        level++;
        score += 100;
        System.out.println("Level: " + level + ", Score: " + score);
    }
    
    public GameMemento save() {
        return new GameMemento(level, score, playerName);
    }
    
    public void load(GameMemento memento) {
        this.level = memento.getLevel();
        this.score = memento.getScore();
        this.playerName = memento.getPlayerName();
        System.out.println("Game loaded - Level: " + level + ", Score: " + score);
    }
    
    public void display() {
        System.out.println("Player: " + playerName + ", Level: " + level + ", Score: " + score);
    }
}

// Caretaker
class GameSaveManager {
    private Map<String, GameMemento> saves;
    
    public GameSaveManager() {
        this.saves = new HashMap<>();
    }
    
    public void saveGame(String saveName, GameMemento memento) {
        saves.put(saveName, memento);
        System.out.println("Game saved as: " + saveName);
    }
    
    public GameMemento loadGame(String saveName) {
        GameMemento memento = saves.get(saveName);
        if (memento != null) {
            System.out.println("Loading game: " + saveName);
        }
        return memento;
    }
    
    public List<String> listSaves() {
        return new ArrayList<>(saves.keySet());
    }
}

// Usage
public class GameMementoExample {
    public static void main(String[] args) {
        Game game = new Game("Player1");
        GameSaveManager saveManager = new GameSaveManager();
        
        game.play();
        game.play();
        saveManager.saveGame("save1", game.save());
        
        game.play();
        game.play();
        game.display();
        
        System.out.println("\nLoading save1:");
        game.load(saveManager.loadGame("save1"));
        game.display();
    }
}
```

---

## 2. State Pattern

### 2.1 Intent

Allow an object to alter its behavior when its internal state changes. The object will appear to have changed its class.

### 2.2 When to Use

- Object behavior depends on its state
- Have many conditional statements based on object state
- State transitions are complex
- Want to avoid state-specific conditionals

### 2.3 Scenarios

- **Vending Machine**: Different states (NoCoin, HasCoin, Sold, SoldOut)
- **Order Processing**: Order states (Pending, Processing, Shipped, Delivered)
- **Media Player**: Play, Pause, Stop states
- **TCP Connection**: Established, Listen, Closed states
- **Workflow Engine**: Task states (New, InProgress, Completed)

### 2.4 Implementation

```java
// State interface
interface State {
    void handleRequest(Context context);
}

// Concrete states
class ConcreteStateA implements State {
    @Override
    public void handleRequest(Context context) {
        System.out.println("Handling request in State A");
        context.setState(new ConcreteStateB());
    }
}

class ConcreteStateB implements State {
    @Override
    public void handleRequest(Context context) {
        System.out.println("Handling request in State B");
        context.setState(new ConcreteStateA());
    }
}

// Context
class Context {
    private State state;
    
    public Context() {
        this.state = new ConcreteStateA();
    }
    
    public void setState(State state) {
        this.state = state;
    }
    
    public void request() {
        state.handleRequest(this);
    }
}
```

### 2.5 Usage Example

```java
public class StateExample {
    public static void main(String[] args) {
        Context context = new Context();
        
        context.request();
        context.request();
        context.request();
    }
}
```

### 2.6 Real-World Example: Vending Machine

```java
// State interface
interface VendingMachineState {
    void insertCoin();
    void ejectCoin();
    void selectProduct();
    void dispense();
}

// Concrete states
class NoCoinState implements VendingMachineState {
    private VendingMachine machine;
    
    public NoCoinState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertCoin() {
        System.out.println("Coin inserted");
        machine.setState(machine.getHasCoinState());
    }
    
    @Override
    public void ejectCoin() {
        System.out.println("No coin to eject");
    }
    
    @Override
    public void selectProduct() {
        System.out.println("Please insert coin first");
    }
    
    @Override
    public void dispense() {
        System.out.println("Please insert coin first");
    }
}

class HasCoinState implements VendingMachineState {
    private VendingMachine machine;
    
    public HasCoinState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertCoin() {
        System.out.println("Coin already inserted");
    }
    
    @Override
    public void ejectCoin() {
        System.out.println("Coin ejected");
        machine.setState(machine.getNoCoinState());
    }
    
    @Override
    public void selectProduct() {
        System.out.println("Product selected");
        machine.setState(machine.getSoldState());
    }
    
    @Override
    public void dispense() {
        System.out.println("Please select product first");
    }
}

class SoldState implements VendingMachineState {
    private VendingMachine machine;
    
    public SoldState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertCoin() {
        System.out.println("Please wait, product is being dispensed");
    }
    
    @Override
    public void ejectCoin() {
        System.out.println("Sorry, product already selected");
    }
    
    @Override
    public void selectProduct() {
        System.out.println("Product already selected");
    }
    
    @Override
    public void dispense() {
        System.out.println("Product dispensed");
        if (machine.getCount() > 0) {
            machine.setState(machine.getNoCoinState());
        } else {
            machine.setState(machine.getSoldOutState());
        }
    }
}

class SoldOutState implements VendingMachineState {
    private VendingMachine machine;
    
    public SoldOutState(VendingMachine machine) {
        this.machine = machine;
    }
    
    @Override
    public void insertCoin() {
        System.out.println("Machine is sold out");
    }
    
    @Override
    public void ejectCoin() {
        System.out.println("No coin inserted");
    }
    
    @Override
    public void selectProduct() {
        System.out.println("Machine is sold out");
    }
    
    @Override
    public void dispense() {
        System.out.println("Machine is sold out");
    }
}

// Context
class VendingMachine {
    private VendingMachineState noCoinState;
    private VendingMachineState hasCoinState;
    private VendingMachineState soldState;
    private VendingMachineState soldOutState;
    
    private VendingMachineState currentState;
    private int count;
    
    public VendingMachine(int count) {
        this.count = count;
        this.noCoinState = new NoCoinState(this);
        this.hasCoinState = new HasCoinState(this);
        this.soldState = new SoldState(this);
        this.soldOutState = new SoldOutState(this);
        
        if (count > 0) {
            this.currentState = noCoinState;
        } else {
            this.currentState = soldOutState;
        }
    }
    
    public void setState(VendingMachineState state) {
        this.currentState = state;
    }
    
    public VendingMachineState getNoCoinState() { return noCoinState; }
    public VendingMachineState getHasCoinState() { return hasCoinState; }
    public VendingMachineState getSoldState() { return soldState; }
    public VendingMachineState getSoldOutState() { return soldOutState; }
    
    public int getCount() { return count; }
    public void setCount(int count) { this.count = count; }
    
    public void insertCoin() { currentState.insertCoin(); }
    public void ejectCoin() { currentState.ejectCoin(); }
    public void selectProduct() { currentState.selectProduct(); }
    public void dispense() { currentState.dispense(); }
}

// Usage
public class VendingMachineExample {
    public static void main(String[] args) {
        VendingMachine machine = new VendingMachine(2);
        
        machine.insertCoin();
        machine.selectProduct();
        machine.dispense();
        
        System.out.println();
        
        machine.insertCoin();
        machine.selectProduct();
        machine.dispense();
        
        System.out.println();
        
        machine.insertCoin(); // Sold out
    }
}
```

### 2.7 Order State Example

```java
// State interface
interface OrderState {
    void process(Order order);
    void cancel(Order order);
    void ship(Order order);
    void deliver(Order order);
}

// Concrete states
class PendingState implements OrderState {
    @Override
    public void process(Order order) {
        System.out.println("Processing order...");
        order.setState(new ProcessingState());
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Order cancelled");
        order.setState(new CancelledState());
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Cannot ship pending order");
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Cannot deliver pending order");
    }
}

class ProcessingState implements OrderState {
    @Override
    public void process(Order order) {
        System.out.println("Order already being processed");
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Cannot cancel order being processed");
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Shipping order...");
        order.setState(new ShippedState());
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Cannot deliver unshipped order");
    }
}

class ShippedState implements OrderState {
    @Override
    public void process(Order order) {
        System.out.println("Order already shipped");
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Cannot cancel shipped order");
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Order already shipped");
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Delivering order...");
        order.setState(new DeliveredState());
    }
}

class DeliveredState implements OrderState {
    @Override
    public void process(Order order) {
        System.out.println("Order already delivered");
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Cannot cancel delivered order");
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Order already delivered");
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Order already delivered");
    }
}

class CancelledState implements OrderState {
    @Override
    public void process(Order order) {
        System.out.println("Cannot process cancelled order");
    }
    
    @Override
    public void cancel(Order order) {
        System.out.println("Order already cancelled");
    }
    
    @Override
    public void ship(Order order) {
        System.out.println("Cannot ship cancelled order");
    }
    
    @Override
    public void deliver(Order order) {
        System.out.println("Cannot deliver cancelled order");
    }
}

// Context
class Order {
    private OrderState state;
    private String orderId;
    
    public Order(String orderId) {
        this.orderId = orderId;
        this.state = new PendingState();
    }
    
    public void setState(OrderState state) {
        this.state = state;
    }
    
    public void process() { state.process(this); }
    public void cancel() { state.cancel(this); }
    public void ship() { state.ship(this); }
    public void deliver() { state.deliver(this); }
}

// Usage
public class OrderStateExample {
    public static void main(String[] args) {
        Order order = new Order("ORD-123");
        
        order.process();
        order.ship();
        order.deliver();
    }
}
```

---

## 3. Template Method Pattern

### 3.1 Intent

Define the skeleton of an algorithm in an operation, deferring some steps to subclasses. Lets subclasses redefine certain steps without changing the algorithm's structure.

### 3.2 When to Use

- Have algorithm with invariant and variant parts
- Want to avoid code duplication
- Want to control algorithm structure
- Subclasses should only extend specific steps

### 3.3 Scenarios

- **Data Processing**: Different data sources, same processing steps
- **Build Process**: Same build steps, different implementations
- **Report Generation**: Same report structure, different data sources
- **Game Framework**: Same game loop, different game logic
- **Framework Design**: Define algorithm skeleton, let users customize

### 3.4 Implementation

```java
// Abstract class with template method
abstract class DataProcessor {
    // Template method - defines algorithm skeleton
    public final void process() {
        readData();
        processData();
        saveData();
    }
    
    // Abstract methods - must be implemented by subclasses
    protected abstract void readData();
    protected abstract void processData();
    
    // Hook method - can be overridden
    protected void saveData() {
        System.out.println("Saving data to default location");
    }
}

// Concrete implementations
class CSVDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from CSV file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data");
    }
    
    @Override
    protected void saveData() {
        System.out.println("Saving processed data to CSV file");
    }
}

class XMLDataProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading data from XML file");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing XML data");
    }
}
```

### 3.5 Usage Example

```java
public class TemplateMethodExample {
    public static void main(String[] args) {
        DataProcessor csvProcessor = new CSVDataProcessor();
        csvProcessor.process();
        
        System.out.println();
        
        DataProcessor xmlProcessor = new XMLDataProcessor();
        xmlProcessor.process();
    }
}
```

### 3.6 Real-World Example: Build Process

```java
// Abstract builder
abstract class BuildProcess {
    // Template method
    public final void build() {
        checkoutCode();
        installDependencies();
        compile();
        runTests();
        packageArtifact();
        deploy();
    }
    
    protected void checkoutCode() {
        System.out.println("Checking out code from repository");
    }
    
    protected abstract void installDependencies();
    
    protected void compile() {
        System.out.println("Compiling code");
    }
    
    protected void runTests() {
        System.out.println("Running tests");
    }
    
    protected abstract void packageArtifact();
    
    protected void deploy() {
        System.out.println("Deploying to default environment");
    }
}

// Concrete builders
class JavaBuildProcess extends BuildProcess {
    @Override
    protected void installDependencies() {
        System.out.println("Installing Maven dependencies");
    }
    
    @Override
    protected void packageArtifact() {
        System.out.println("Packaging JAR file");
    }
}

class NodeBuildProcess extends BuildProcess {
    @Override
    protected void installDependencies() {
        System.out.println("Installing npm packages");
    }
    
    @Override
    protected void packageArtifact() {
        System.out.println("Packaging Node.js application");
    }
    
    @Override
    protected void deploy() {
        System.out.println("Deploying to Node.js server");
    }
}

// Usage
public class BuildProcessExample {
    public static void main(String[] args) {
        BuildProcess javaBuild = new JavaBuildProcess();
        System.out.println("=== Java Build ===");
        javaBuild.build();
        
        System.out.println("\n=== Node.js Build ===");
        BuildProcess nodeBuild = new NodeBuildProcess();
        nodeBuild.build();
    }
}
```

### 3.7 Game Framework Example

```java
// Abstract game
abstract class Game {
    // Template method
    public final void play() {
        initialize();
        startPlay();
        endPlay();
    }
    
    protected abstract void initialize();
    protected abstract void startPlay();
    protected abstract void endPlay();
}

// Concrete games
class Chess extends Game {
    @Override
    protected void initialize() {
        System.out.println("Chess: Setting up board and pieces");
    }
    
    @Override
    protected void startPlay() {
        System.out.println("Chess: Starting game");
    }
    
    @Override
    protected void endPlay() {
        System.out.println("Chess: Game ended");
    }
}

class Football extends Game {
    @Override
    protected void initialize() {
        System.out.println("Football: Setting up field and teams");
    }
    
    @Override
    protected void startPlay() {
        System.out.println("Football: Match started");
    }
    
    @Override
    protected void endPlay() {
        System.out.println("Football: Match ended");
    }
}

// Usage
public class GameFrameworkExample {
    public static void main(String[] args) {
        Game chess = new Chess();
        chess.play();
        
        System.out.println();
        
        Game football = new Football();
        football.play();
    }
}
```

---

## Summary: Part 7

### Patterns Covered

1. **Memento**: Save and restore object state
2. **State**: Change behavior based on state
3. **Template Method**: Define algorithm skeleton

### Key Takeaways

- **Memento**: Use for undo/redo and state restoration
- **State**: Use when behavior depends on state
- **Template Method**: Use to define algorithm structure

### Next Steps

**Part 8** will cover:
- Visitor Pattern
- Interpreter Pattern
- Null Object Pattern

---

**Master these behavioral patterns for flexible algorithm design!**

