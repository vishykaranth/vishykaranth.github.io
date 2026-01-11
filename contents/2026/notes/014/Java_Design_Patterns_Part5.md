# Java Design Patterns: Complete Guide with Examples

## Part 5: Behavioral Patterns - Observer, Strategy, Command

---

## Table of Contents

1. [Observer Pattern](#1-observer-pattern)
2. [Strategy Pattern](#2-strategy-pattern)
3. [Command Pattern](#3-command-pattern)

---

## 1. Observer Pattern

### 1.1 Intent

Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

### 1.2 When to Use

- Change to one object requires changing other objects
- Number of dependent objects is unknown
- Objects should be loosely coupled
- Need to notify multiple objects about state changes

### 1.3 Scenarios

- **Event Handling**: GUI events, button clicks
- **Model-View**: MVC architecture
- **Stock Market**: Notify investors of price changes
- **News Publisher**: Notify subscribers of news
- **Weather Station**: Notify displays of weather changes

### 1.4 Implementation

```java
import java.util.ArrayList;
import java.util.List;

// Subject interface
interface Subject {
    void registerObserver(Observer observer);
    void removeObserver(Observer observer);
    void notifyObservers();
}

// Observer interface
interface Observer {
    void update(String message);
}

// Concrete subject
class NewsAgency implements Subject {
    private List<Observer> observers;
    private String news;
    
    public NewsAgency() {
        this.observers = new ArrayList<>();
    }
    
    @Override
    public void registerObserver(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void removeObserver(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

// Concrete observers
class NewsChannel implements Observer {
    private String name;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}
```

### 1.5 Usage Example

```java
public class ObserverExample {
    public static void main(String[] args) {
        NewsAgency agency = new NewsAgency();
        
        NewsChannel channel1 = new NewsChannel("CNN");
        NewsChannel channel2 = new NewsChannel("BBC");
        NewsChannel channel3 = new NewsChannel("Fox News");
        
        agency.registerObserver(channel1);
        agency.registerObserver(channel2);
        agency.registerObserver(channel3);
        
        agency.setNews("Breaking: Major event happened!");
        
        System.out.println();
        agency.removeObserver(channel2);
        agency.setNews("Update: More details available");
    }
}
```

### 1.6 Real-World Example: Stock Market

```java
import java.util.*;

// Subject
class StockMarket implements Subject {
    private List<Observer> investors;
    private Map<String, Double> stockPrices;
    
    public StockMarket() {
        this.investors = new ArrayList<>();
        this.stockPrices = new HashMap<>();
    }
    
    @Override
    public void registerObserver(Observer observer) {
        investors.add(observer);
    }
    
    @Override
    public void removeObserver(Observer observer) {
        investors.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer investor : investors) {
            investor.update(stockPrices);
        }
    }
    
    public void updateStockPrice(String symbol, double price) {
        stockPrices.put(symbol, price);
        notifyObservers();
    }
}

// Observer
class Investor implements Observer {
    private String name;
    private List<String> watchedStocks;
    
    public Investor(String name, List<String> watchedStocks) {
        this.name = name;
        this.watchedStocks = watchedStocks;
    }
    
    @Override
    public void update(Map<String, Double> stockPrices) {
        System.out.println(name + " received stock update:");
        for (String stock : watchedStocks) {
            Double price = stockPrices.get(stock);
            if (price != null) {
                System.out.println("  " + stock + ": $" + price);
            }
        }
    }
}

// Usage
public class StockMarketExample {
    public static void main(String[] args) {
        StockMarket market = new StockMarket();
        
        Investor investor1 = new Investor("John", Arrays.asList("AAPL", "GOOGL"));
        Investor investor2 = new Investor("Jane", Arrays.asList("GOOGL", "MSFT"));
        
        market.registerObserver(investor1);
        market.registerObserver(investor2);
        
        market.updateStockPrice("AAPL", 150.0);
        market.updateStockPrice("GOOGL", 2500.0);
        market.updateStockPrice("MSFT", 300.0);
    }
}
```

### 1.7 Java Built-in Observer (Deprecated but shows concept)

```java
import java.util.Observable;
import java.util.Observer;

// Subject (using Observable)
class WeatherStation extends Observable {
    private float temperature;
    private float humidity;
    
    public void setMeasurements(float temperature, float humidity) {
        this.temperature = temperature;
        this.humidity = humidity;
        setChanged();
        notifyObservers();
    }
    
    public float getTemperature() {
        return temperature;
    }
    
    public float getHumidity() {
        return humidity;
    }
}

// Observer
class WeatherDisplay implements Observer {
    private String name;
    
    public WeatherDisplay(String name) {
        this.name = name;
    }
    
    @Override
    public void update(Observable o, Object arg) {
        if (o instanceof WeatherStation) {
            WeatherStation station = (WeatherStation) o;
            System.out.println(name + " - Temperature: " + station.getTemperature() + 
                             "°F, Humidity: " + station.getHumidity() + "%");
        }
    }
}

// Usage
public class JavaObserverExample {
    public static void main(String[] args) {
        WeatherStation station = new WeatherStation();
        
        WeatherDisplay display1 = new WeatherDisplay("Display 1");
        WeatherDisplay display2 = new WeatherDisplay("Display 2");
        
        station.addObserver(display1);
        station.addObserver(display2);
        
        station.setMeasurements(75.0f, 65.0f);
    }
}
```

---

## 2. Strategy Pattern

### 2.1 Intent

Define a family of algorithms, encapsulate each one, and make them interchangeable. Strategy lets the algorithm vary independently from clients that use it.

### 2.2 When to Use

- Have multiple ways to perform a task
- Want to avoid conditional statements for algorithm selection
- Algorithms should be interchangeable
- Want to isolate algorithm implementation from client code

### 2.3 Scenarios

- **Payment Processing**: Different payment methods (Credit Card, PayPal, Bank Transfer)
- **Sorting Algorithms**: Different sorting strategies (QuickSort, MergeSort, BubbleSort)
- **Compression**: Different compression algorithms (ZIP, RAR, 7Z)
- **Validation**: Different validation strategies
- **Discount Calculation**: Different discount types

### 2.4 Implementation

```java
// Strategy interface
interface PaymentStrategy {
    void pay(double amount);
}

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    private String name;
    
    public CreditCardPayment(String cardNumber, String name) {
        this.cardNumber = cardNumber;
        this.name = name;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using Credit Card ending in " + 
                          cardNumber.substring(cardNumber.length() - 4));
    }
}

class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " using PayPal account: " + email);
    }
}

class BankTransferPayment implements PaymentStrategy {
    private String accountNumber;
    
    public BankTransferPayment(String accountNumber) {
        this.accountNumber = accountNumber;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid $" + amount + " via Bank Transfer from account: " + accountNumber);
    }
}

// Context
class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }
    
    public void checkout(double amount) {
        paymentStrategy.pay(amount);
    }
}
```

### 2.5 Usage Example

```java
public class StrategyExample {
    public static void main(String[] args) {
        ShoppingCart cart = new ShoppingCart();
        
        // Pay with credit card
        cart.setPaymentStrategy(new CreditCardPayment("1234567890123456", "John Doe"));
        cart.checkout(100.0);
        
        // Pay with PayPal
        cart.setPaymentStrategy(new PayPalPayment("john@example.com"));
        cart.checkout(200.0);
        
        // Pay with bank transfer
        cart.setPaymentStrategy(new BankTransferPayment("ACC123456"));
        cart.checkout(300.0);
    }
}
```

### 2.6 Real-World Example: Sorting Strategy

```java
import java.util.Arrays;
import java.util.List;

// Strategy interface
interface SortStrategy {
    void sort(List<Integer> list);
}

// Concrete strategies
class BubbleSortStrategy implements SortStrategy {
    @Override
    public void sort(List<Integer> list) {
        System.out.println("Sorting using Bubble Sort");
        // Bubble sort implementation
        Integer[] arr = list.toArray(new Integer[0]);
        int n = arr.length;
        for (int i = 0; i < n - 1; i++) {
            for (int j = 0; j < n - i - 1; j++) {
                if (arr[j] > arr[j + 1]) {
                    int temp = arr[j];
                    arr[j] = arr[j + 1];
                    arr[j + 1] = temp;
                }
            }
        }
        list.clear();
        list.addAll(Arrays.asList(arr));
    }
}

class QuickSortStrategy implements SortStrategy {
    @Override
    public void sort(List<Integer> list) {
        System.out.println("Sorting using Quick Sort");
        // Quick sort implementation
        Integer[] arr = list.toArray(new Integer[0]);
        quickSort(arr, 0, arr.length - 1);
        list.clear();
        list.addAll(Arrays.asList(arr));
    }
    
    private void quickSort(Integer[] arr, int low, int high) {
        if (low < high) {
            int pi = partition(arr, low, high);
            quickSort(arr, low, pi - 1);
            quickSort(arr, pi + 1, high);
        }
    }
    
    private int partition(Integer[] arr, int low, int high) {
        int pivot = arr[high];
        int i = low - 1;
        for (int j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                int temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
        int temp = arr[i + 1];
        arr[i + 1] = arr[high];
        arr[high] = temp;
        return i + 1;
    }
}

class MergeSortStrategy implements SortStrategy {
    @Override
    public void sort(List<Integer> list) {
        System.out.println("Sorting using Merge Sort");
        // Merge sort implementation
        Integer[] arr = list.toArray(new Integer[0]);
        mergeSort(arr, 0, arr.length - 1);
        list.clear();
        list.addAll(Arrays.asList(arr));
    }
    
    private void mergeSort(Integer[] arr, int left, int right) {
        if (left < right) {
            int mid = left + (right - left) / 2;
            mergeSort(arr, left, mid);
            mergeSort(arr, mid + 1, right);
            merge(arr, left, mid, right);
        }
    }
    
    private void merge(Integer[] arr, int left, int mid, int right) {
        int n1 = mid - left + 1;
        int n2 = right - mid;
        
        Integer[] leftArr = new Integer[n1];
        Integer[] rightArr = new Integer[n2];
        
        System.arraycopy(arr, left, leftArr, 0, n1);
        System.arraycopy(arr, mid + 1, rightArr, 0, n2);
        
        int i = 0, j = 0, k = left;
        while (i < n1 && j < n2) {
            if (leftArr[i] <= rightArr[j]) {
                arr[k++] = leftArr[i++];
            } else {
                arr[k++] = rightArr[j++];
            }
        }
        
        while (i < n1) arr[k++] = leftArr[i++];
        while (j < n2) arr[k++] = rightArr[j++];
    }
}

// Context
class Sorter {
    private SortStrategy strategy;
    
    public void setStrategy(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sort(List<Integer> list) {
        strategy.sort(list);
    }
}

// Usage
public class SortStrategyExample {
    public static void main(String[] args) {
        Sorter sorter = new Sorter();
        List<Integer> numbers = Arrays.asList(64, 34, 25, 12, 22, 11, 90);
        
        // Use bubble sort
        List<Integer> list1 = new ArrayList<>(numbers);
        sorter.setStrategy(new BubbleSortStrategy());
        sorter.sort(list1);
        System.out.println("Sorted: " + list1);
        
        // Use quick sort
        List<Integer> list2 = new ArrayList<>(numbers);
        sorter.setStrategy(new QuickSortStrategy());
        sorter.sort(list2);
        System.out.println("Sorted: " + list2);
        
        // Use merge sort
        List<Integer> list3 = new ArrayList<>(numbers);
        sorter.setStrategy(new MergeSortStrategy());
        sorter.sort(list3);
        System.out.println("Sorted: " + list3);
    }
}
```

---

## 3. Command Pattern

### 3.1 Intent

Encapsulate a request as an object, thereby allowing parameterization of clients with different requests, queuing of requests, and logging of requests.

### 3.2 When to Use

- Need to parameterize objects with operations
- Need to queue operations, schedule, or execute at different times
- Need to support undo/redo operations
- Need to log operations
- Need to support transactions

### 3.3 Scenarios

- **Remote Control**: Different buttons execute different commands
- **Text Editor**: Undo/redo operations
- **Job Queue**: Queue and execute jobs
- **Macro Recording**: Record and replay commands
- **Transaction Management**: Execute and rollback operations

### 3.4 Implementation

```java
// Command interface
interface Command {
    void execute();
    void undo();
}

// Receiver
class Light {
    public void turnOn() {
        System.out.println("Light is ON");
    }
    
    public void turnOff() {
        System.out.println("Light is OFF");
    }
}

// Concrete commands
class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOn();
    }
    
    @Override
    public void undo() {
        light.turnOff();
    }
}

class LightOffCommand implements Command {
    private Light light;
    
    public LightOffCommand(Light light) {
        this.light = light;
    }
    
    @Override
    public void execute() {
        light.turnOff();
    }
    
    @Override
    public void undo() {
        light.turnOn();
    }
}

// Invoker
class RemoteControl {
    private Command command;
    private Command lastCommand;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        if (command != null) {
            command.execute();
            lastCommand = command;
        }
    }
    
    public void pressUndo() {
        if (lastCommand != null) {
            lastCommand.undo();
        }
    }
}
```

### 3.5 Usage Example

```java
public class CommandExample {
    public static void main(String[] args) {
        Light light = new Light();
        
        Command lightOn = new LightOnCommand(light);
        Command lightOff = new LightOffCommand(light);
        
        RemoteControl remote = new RemoteControl();
        
        // Turn on light
        remote.setCommand(lightOn);
        remote.pressButton();
        
        // Turn off light
        remote.setCommand(lightOff);
        remote.pressButton();
        
        // Undo last command
        remote.pressUndo();
    }
}
```

### 3.6 Real-World Example: Text Editor with Undo/Redo

```java
import java.util.Stack;

// Receiver
class TextEditor {
    private StringBuilder text;
    
    public TextEditor() {
        this.text = new StringBuilder();
    }
    
    public void insert(String text) {
        this.text.append(text);
        System.out.println("Text inserted. Current text: " + this.text);
    }
    
    public void delete(int length) {
        if (text.length() >= length) {
            text.delete(text.length() - length, text.length());
            System.out.println("Text deleted. Current text: " + text);
        }
    }
    
    public String getText() {
        return text.toString();
    }
}

// Command interface
interface TextCommand {
    void execute();
    void undo();
}

// Concrete commands
class InsertCommand implements TextCommand {
    private TextEditor editor;
    private String text;
    
    public InsertCommand(TextEditor editor, String text) {
        this.editor = editor;
        this.text = text;
    }
    
    @Override
    public void execute() {
        editor.insert(text);
    }
    
    @Override
    public void undo() {
        editor.delete(text.length());
    }
}

class DeleteCommand implements TextCommand {
    private TextEditor editor;
    private int length;
    private String deletedText;
    
    public DeleteCommand(TextEditor editor, int length) {
        this.editor = editor;
        this.length = length;
    }
    
    @Override
    public void execute() {
        String currentText = editor.getText();
        if (currentText.length() >= length) {
            deletedText = currentText.substring(currentText.length() - length);
            editor.delete(length);
        }
    }
    
    @Override
    public void undo() {
        if (deletedText != null) {
            editor.insert(deletedText);
        }
    }
}

// Invoker with undo/redo
class TextEditorInvoker {
    private Stack<TextCommand> undoStack;
    private Stack<TextCommand> redoStack;
    
    public TextEditorInvoker() {
        this.undoStack = new Stack<>();
        this.redoStack = new Stack<>();
    }
    
    public void executeCommand(TextCommand command) {
        command.execute();
        undoStack.push(command);
        redoStack.clear(); // Clear redo stack when new command executed
    }
    
    public void undo() {
        if (!undoStack.isEmpty()) {
            TextCommand command = undoStack.pop();
            command.undo();
            redoStack.push(command);
        }
    }
    
    public void redo() {
        if (!redoStack.isEmpty()) {
            TextCommand command = redoStack.pop();
            command.execute();
            undoStack.push(command);
        }
    }
}

// Usage
public class TextEditorCommandExample {
    public static void main(String[] args) {
        TextEditor editor = new TextEditor();
        TextEditorInvoker invoker = new TextEditorInvoker();
        
        // Insert text
        invoker.executeCommand(new InsertCommand(editor, "Hello "));
        invoker.executeCommand(new InsertCommand(editor, "World"));
        
        // Undo
        System.out.println("\nUndoing last command:");
        invoker.undo();
        
        // Redo
        System.out.println("\nRedoing:");
        invoker.redo();
        
        // Delete
        System.out.println("\nDeleting:");
        invoker.executeCommand(new DeleteCommand(editor, 6));
        
        // Undo delete
        System.out.println("\nUndoing delete:");
        invoker.undo();
    }
}
```

### 3.7 Macro Command Example

```java
import java.util.*;

// Macro command (composite of commands)
class MacroCommand implements Command {
    private List<Command> commands;
    
    public MacroCommand(List<Command> commands) {
        this.commands = commands;
    }
    
    @Override
    public void execute() {
        for (Command command : commands) {
            command.execute();
        }
    }
    
    @Override
    public void undo() {
        // Undo in reverse order
        for (int i = commands.size() - 1; i >= 0; i--) {
            commands.get(i).undo();
        }
    }
}

// Usage
public class MacroCommandExample {
    public static void main(String[] args) {
        Light light1 = new Light();
        Light light2 = new Light();
        
        List<Command> commands = Arrays.asList(
            new LightOnCommand(light1),
            new LightOnCommand(light2)
        );
        
        MacroCommand macro = new MacroCommand(commands);
        
        RemoteControl remote = new RemoteControl();
        remote.setCommand(macro);
        remote.pressButton();
        
        System.out.println("\nUndoing macro:");
        remote.pressUndo();
    }
}
```

---

## Summary: Part 5

### Patterns Covered

1. **Observer**: One-to-many dependency for notifications
2. **Strategy**: Encapsulate interchangeable algorithms
3. **Command**: Encapsulate requests as objects

### Key Takeaways

- **Observer**: Use for event-driven systems and notifications
- **Strategy**: Use when algorithms are interchangeable
- **Command**: Use for undo/redo, queuing, and logging operations

### Next Steps

**Part 6** will cover:
- Chain of Responsibility Pattern
- Iterator Pattern
- Mediator Pattern

---

**Master these behavioral patterns for flexible object communication!**

