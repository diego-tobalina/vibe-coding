---
name: java-design-patterns
description: Comprehensive Java design patterns reference. Use when implementing creational, structural, or behavioral patterns.
allowed-tools:
  - Read
  - Grep
  - Glob
  - Edit
  - Write
---

# Java Design Patterns

Quick reference for all Gang of Four design patterns in Java.

## Pattern Selection Guide

### Creational Patterns (Object Creation)
| Pattern | Use When | Don't Use When |
|---------|----------|----------------|
| **Singleton** | Exactly one instance needed (DB connection, cache) | Multiple instances needed, testing is hard |
| **Factory Method** | Delegate instantiation to subclasses | Single product type, simple creation |
| **Abstract Factory** | Create families of related objects | Single products, product types change often |
| **Builder** | Many optional params, step-by-step construction | Few params, simple objects |
| **Prototype** | Creating objects is expensive, clone existing | Objects are simple, deep cloning is complex |

### Structural Patterns (Object Composition)
| Pattern | Use When | Don't Use When |
|---------|----------|----------------|
| **Adapter** | Interface incompatible with client | Interfaces already compatible |
| **Bridge** | Abstraction and implementation vary independently | Simple hierarchy, few variations |
| **Composite** | Tree structure, treat individual and groups uniformly | No hierarchy, leaf-only structure |
| **Decorator** | Add responsibilities dynamically | Many decorators cause confusion |
| **Facade** | Simplify complex subsystem interface | Simple subsystem, direct access needed |
| **Flyweight** | Many similar objects, share state to save memory | Few objects, distinct states |
| **Proxy** | Control access, lazy loading, logging | Simple object access, adds unnecessary complexity |

### Behavioral Patterns (Object Interaction)
| Pattern | Use When | Don't Use When |
|---------|----------|----------------|
| **Chain of Responsibility** | Multiple handlers, dynamic chain | Chain is fixed, simple handling |
| **Command** | Queue/undo operations, decouple sender/receiver | Simple direct calls, no undo needed |
| **Iterator** | Traverse collection without exposing internals | Simple collections, Java's Iterator sufficient |
| **Mediator** | Complex communication between many objects | Few objects, simple interactions |
| **Memento** | Save/restore object state (undo) | State is simple, or persistence not needed |
| **Observer** | One-to-many dependency, event subscription | No notification needed, tight coupling OK |
| **State** | Object behavior changes based on state | Few states, simple conditions suffice |
| **Strategy** | Multiple algorithms, interchangeable | Single algorithm, no variation |
| **Template Method** | Common algorithm with variant steps | Algorithm varies completely |
| **Visitor** | Separate operations from object structure | Structure changes often, few operations |

---

## Creational Patterns

### 1. Singleton Pattern
Ensure a class has only one instance with global access.

```java
// Thread-Safe Singleton with Double-Checked Locking
public final class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    
    private DatabaseConnection() {
        // private constructor
    }
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}

// Alternative: Enum Singleton (serialization-safe, reflection-safe)
public enum EnumSingleton {
    INSTANCE;
    
    public void doSomething() {
        // business logic
    }
}
```

**Use for:** DB connections, caches, thread pools, configuration

---

### 2. Factory Method Pattern
Create objects without specifying exact class.

```java
// Product Interface
public interface Notification {
    void send(String message);
}

// Concrete Products
public class EmailNotification implements Notification {
    public void send(String message) {
        System.out.println("Email: " + message);
    }
}

public class SmsNotification implements Notification {
    public void send(String message) {
        System.out.println("SMS: " + message);
    }
}

// Creator
public abstract class NotificationFactory {
    abstract Notification createNotification();
    
    public void notifyUser(String message) {
        Notification notification = createNotification();
        notification.send(message);
    }
}

// Concrete Creators
public class EmailNotificationFactory extends NotificationFactory {
    Notification createNotification() {
        return new EmailNotification();
    }
}

// Usage
NotificationFactory factory = new EmailNotificationFactory();
factory.notifyUser("Hello!");
```

---

### 3. Abstract Factory Pattern
Create families of related objects.

```java
// Abstract Factory
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete Factories
public class WindowsFactory implements GUIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}

public class MacFactory implements GUIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}

// Usage
GUIFactory factory = System.getProperty("os.name").contains("Windows") 
    ? new WindowsFactory() : new MacFactory();
Button button = factory.createButton();
```

---

### 4. Builder Pattern
Construct complex objects step by step.

```java
// With Lombok (Recommended)
@Data
@Builder
public class User {
    private String id;
    private String email;
    private String name;
    private int age;
    private String address;
}

// Usage
User user = User.builder()
    .id("123")
    .email("user@example.com")
    .name("John")
    .build();

// Manual Implementation
public class UserBuilder {
    private User user = new User();
    
    public UserBuilder withEmail(String email) {
        user.setEmail(email);
        return this;
    }
    
    public UserBuilder withName(String name) {
        user.setName(name);
        return this;
    }
    
    public User build() {
        // validation
        if (user.getEmail() == null) {
            throw new IllegalStateException("Email is required");
        }
        return user;
    }
}
```

---

### 5. Prototype Pattern
Clone existing objects.

```java
public interface Prototype<T> extends Cloneable {
    T clone();
}

public class Document implements Prototype<Document> {
    private String title;
    private String content;
    private List<String> tags;
    
    // Deep copy
    @Override
    public Document clone() {
        try {
            Document cloned = (Document) super.clone();
            cloned.tags = new ArrayList<>(this.tags); // deep copy list
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

// Usage
Document template = new Document("Template", "Content", Arrays.asList("tag1"));
Document copy = template.clone();
```

---

## Structural Patterns

### 6. Adapter Pattern
Convert interface to another expected by client.

```java
// Target interface
public interface MediaPlayer {
    void play(String filename);
}

// Adaptee (existing interface)
public interface AdvancedMediaPlayer {
    void playMp4(String filename);
    void playVlc(String filename);
}

// Adapter
public class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;
    
    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer = new Mp4Player();
        }
    }
    
    public void play(String filename) {
        advancedPlayer.playMp4(filename);
    }
}
```

---

### 7. Bridge Pattern
Separate abstraction from implementation.

```java
// Implementation interface
public interface DrawingAPI {
    void drawCircle(double x, double y, double radius);
}

// Concrete Implementations
public class DrawingAPI1 implements DrawingAPI {
    public void drawCircle(double x, double y, double r) {
        System.out.printf("API1.circle at (%.2f,%.2f) radius %.2f%n", x, y, r);
    }
}

// Abstraction
public abstract class Shape {
    protected DrawingAPI drawingAPI;
    
    protected Shape(DrawingAPI api) {
        this.drawingAPI = api;
    }
    
    public abstract void draw();
}

// Refined Abstraction
public class CircleShape extends Shape {
    private double x, y, radius;
    
    public CircleShape(double x, double y, double r, DrawingAPI api) {
        super(api);
        this.x = x; this.y = y; this.radius = r;
    }
    
    public void draw() {
        drawingAPI.drawCircle(x, y, radius);
    }
}

// Usage
Shape circle = new CircleShape(1, 2, 3, new DrawingAPI1());
circle.draw();
```

---

### 8. Composite Pattern
Compose objects into tree structures.

```java
// Component
public interface Employee {
    void showEmployeeDetails();
    double getSalary();
}

// Leaf
public class Developer implements Employee {
    private String name;
    private double salary;
    
    public void showEmployeeDetails() {
        System.out.println(name + " - " + salary);
    }
    
    public double getSalary() { return salary; }
}

// Composite
public class Manager implements Employee {
    private String name;
    private double salary;
    private List<Employee> employees = new ArrayList<>();
    
    public void addEmployee(Employee emp) {
        employees.add(emp);
    }
    
    public void showEmployeeDetails() {
        System.out.println(name + " - " + salary);
        for (Employee emp : employees) {
            emp.showEmployeeDetails();
        }
    }
    
    public double getSalary() {
        return salary + employees.stream().mapToDouble(Employee::getSalary).sum();
    }
}
```

---

### 9. Decorator Pattern
Add responsibilities dynamically.

```java
// Component
public interface Coffee {
    double getCost();
    String getDescription();
}

// Concrete Component
public class SimpleCoffee implements Coffee {
    public double getCost() { return 10; }
    public String getDescription() { return "Simple coffee"; }
}

// Decorator
public abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.decoratedCoffee = coffee;
    }
    
    public double getCost() {
        return decoratedCoffee.getCost();
    }
    
    public String getDescription() {
        return decoratedCoffee.getDescription();
    }
}

// Concrete Decorators
public class Milk extends CoffeeDecorator {
    public Milk(Coffee coffee) { super(coffee); }
    
    public double getCost() { return super.getCost() + 2; }
    public String getDescription() { return super.getDescription() + ", milk"; }
}

// Usage
Coffee coffee = new SimpleCoffee();
coffee = new Milk(coffee);
coffee = new Sugar(coffee);
System.out.println(coffee.getDescription() + " $" + coffee.getCost());
```

---

### 10. Facade Pattern
Simplified interface to complex subsystem.

```java
public class ComputerFacade {
    private CPU cpu;
    private Memory memory;
    private HardDrive hardDrive;
    
    public ComputerFacade() {
        this.cpu = new CPU();
        this.memory = new Memory();
        this.hardDrive = new HardDrive();
    }
    
    public void start() {
        cpu.freeze();
        memory.load(0, hardDrive.read(0, 1024));
        cpu.jump(0);
        cpu.execute();
    }
    
    public void shutdown() {
        cpu.stop();
        memory.clear();
    }
}

// Usage - simple interface hides complexity
ComputerFacade computer = new ComputerFacade();
computer.start();
```

---

### 11. Flyweight Pattern
Share data to support large numbers of objects.

```java
// Flyweight
public interface Character {
    void display(int position);
}

// Concrete Flyweight (intrinsic state)
public class CharacterGlyph implements Character {
    private char symbol; // shared
    private String font; // shared
    
    public CharacterGlyph(char symbol, String font) {
        this.symbol = symbol;
        this.font = font;
    }
    
    public void display(int position) { // extrinsic state
        System.out.println(symbol + " at position " + position);
    }
}

// Flyweight Factory
public class CharacterFactory {
    private Map<String, Character> characters = new HashMap<>();
    
    public Character getCharacter(char symbol, String font) {
        String key = symbol + font;
        return characters.computeIfAbsent(key, 
            k -> new CharacterGlyph(symbol, font));
    }
}
```

---

### 12. Proxy Pattern
Placeholder to control access.

```java
// Subject
public interface Image {
    void display();
}

// Real Subject
public class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading " + filename);
    }
    
    public void display() {
        System.out.println("Displaying " + filename);
    }
}

// Proxy
public class ProxyImage implements Image {
    private RealImage realImage;
    private String filename;
    
    public ProxyImage(String filename) {
        this.filename = filename;
    }
    
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename); // lazy loading
        }
        realImage.display();
    }
}

// Usage
Image image = new ProxyImage("photo.jpg"); // no loading yet
image.display(); // loads and displays
```

---

## Behavioral Patterns

### 13. Chain of Responsibility Pattern
Pass requests along chain of handlers.

```java
public abstract class Logger {
    public static int INFO = 1;
    public static int DEBUG = 2;
    public static int ERROR = 3;
    
    protected int level;
    protected Logger nextLogger;
    
    public void setNextLogger(Logger next) {
        this.nextLogger = next;
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

// Concrete Handlers
public class ConsoleLogger extends Logger {
    public ConsoleLogger(int level) { this.level = level; }
    protected void write(String message) {
        System.out.println("Console: " + message);
    }
}

// Usage - chain them together
Logger loggerChain = new ConsoleLogger(Logger.INFO);
Logger errorLogger = new ErrorLogger(Logger.ERROR);
loggerChain.setNextLogger(errorLogger);

loggerChain.logMessage(Logger.INFO, "Info message");
```

---

### 14. Command Pattern
Encapsulate requests as objects.

```java
// Command
public interface Command {
    void execute();
    void undo();
}

// Receiver
public class Light {
    public void on() { System.out.println("Light is on"); }
    public void off() { System.out.println("Light is off"); }
}

// Concrete Command
public class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) { this.light = light; }
    
    public void execute() { light.on(); }
    public void undo() { light.off(); }
}

// Invoker
public class RemoteControl {
    private Command command;
    
    public void setCommand(Command cmd) { this.command = cmd; }
    public void pressButton() { command.execute(); }
    public void pressUndo() { command.undo(); }
}

// Usage
Light light = new Light();
Command lightOn = new LightOnCommand(light);

RemoteControl remote = new RemoteControl();
remote.setCommand(lightOn);
remote.pressButton();
```

---

### 15. Iterator Pattern
Traverse collection without exposing internals.

```java
// Iterator
public interface Iterator<T> {
    boolean hasNext();
    T next();
}

// Aggregate
public interface Container<T> {
    Iterator<T> getIterator();
}

// Concrete Collection
public class NameRepository implements Container<String> {
    private String[] names = {"Robert", "John", "Julie"};
    
    public Iterator<String> getIterator() {
        return new NameIterator();
    }
    
    private class NameIterator implements Iterator<String> {
        private int index;
        
        public boolean hasNext() {
            return index < names.length;
        }
        
        public String next() {
            return hasNext() ? names[index++] : null;
        }
    }
}

// Usage - Java's enhanced for-loop uses Iterator internally
NameRepository repo = new NameRepository();
for (Iterator<String> it = repo.getIterator(); it.hasNext(); ) {
    System.out.println(it.next());
}
```

---

### 16. Mediator Pattern
Encapsulate how objects interact.

```java
// Mediator
public interface ChatMediator {
    void sendMessage(String msg, User user);
    void addUser(User user);
}

// Concrete Mediator
public class ChatMediatorImpl implements ChatMediator {
    private List<User> users = new ArrayList<>();
    
    public void addUser(User user) {
        users.add(user);
    }
    
    public void sendMessage(String msg, User sender) {
        for (User user : users) {
            if (user != sender) {
                user.receive(msg);
            }
        }
    }
}

// Colleague
public abstract class User {
    protected ChatMediator mediator;
    protected String name;
    
    public User(ChatMediator med, String name) {
        this.mediator = med;
        this.name = name;
    }
    
    public abstract void send(String msg);
    public abstract void receive(String msg);
}

// Concrete Colleague
public class UserImpl extends User {
    public UserImpl(ChatMediator med, String name) {
        super(med, name);
    }
    
    public void send(String msg) {
        System.out.println(name + ": Sending Message=" + msg);
        mediator.sendMessage(msg, this);
    }
    
    public void receive(String msg) {
        System.out.println(name + ": Received Message=" + msg);
    }
}

// Usage
ChatMediator mediator = new ChatMediatorImpl();
User user1 = new UserImpl(mediator, "John");
User user2 = new UserImpl(mediator, "Jane");

mediator.addUser(user1);
mediator.addUser(user2);

user1.send("Hi Jane!");
```

---

### 17. Memento Pattern
Capture and restore object state.

```java
// Memento
public class EditorMemento {
    private final String content;
    private final LocalDateTime savedAt;
    
    public EditorMemento(String content) {
        this.content = content;
        this.savedAt = LocalDateTime.now();
    }
    
    public String getContent() { return content; }
}

// Originator
public class Editor {
    private String content;
    
    public void setContent(String content) {
        this.content = content;
    }
    
    public String getContent() { return content; }
    
    public EditorMemento save() {
        return new EditorMemento(content);
    }
    
    public void restore(EditorMemento memento) {
        content = memento.getContent();
    }
}

// Caretaker
public class History {
    private List<EditorMemento> mementos = new ArrayList<>();
    private Editor editor;
    
    public History(Editor editor) {
        this.editor = editor;
    }
    
    public void backup() {
        mementos.add(editor.save());
    }
    
    public void undo() {
        if (!mementos.isEmpty()) {
            EditorMemento memento = mementos.remove(mementos.size() - 1);
            editor.restore(memento);
        }
    }
}

// Usage
Editor editor = new Editor();
History history = new History(editor);

editor.setContent("Version 1");
history.backup();

editor.setContent("Version 2");
history.undo(); // Restores to "Version 1"
```

---

### 18. Observer Pattern
Define one-to-many dependency.

```java
// Observer
public interface StockObserver {
    void update(String stockSymbol, double price);
}

// Subject
public interface StockSubject {
    void registerObserver(StockObserver observer);
    void removeObserver(StockObserver observer);
    void notifyObservers();
}

// Concrete Subject
public class StockPrice implements StockSubject {
    private List<StockObserver> observers = new ArrayList<>();
    private String symbol;
    private double price;
    
    public void setPrice(double price) {
        this.price = price;
        notifyObservers();
    }
    
    public void registerObserver(StockObserver observer) {
        observers.add(observer);
    }
    
    public void notifyObservers() {
        for (StockObserver observer : observers) {
            observer.update(symbol, price);
        }
    }
}

// Concrete Observer
public class MobileApp implements StockObserver {
    public void update(String symbol, double price) {
        System.out.println("MobileApp: " + symbol + " is now $" + price);
    }
}

// Usage with Java's built-in (java.util.Observer is deprecated, use PropertyChangeListener)
StockPrice stock = new StockPrice();
stock.registerObserver(new MobileApp());
stock.registerObserver(new WebApp());

stock.setPrice(150.00); // Notifies all observers
```

---

### 19. State Pattern
Alter behavior when state changes.

```java
// State
public interface VendingMachineState {
    void insertMoney();
    void ejectMoney();
    void dispense();
}

// Concrete States
public class NoMoneyState implements VendingMachineState {
    private VendingMachine machine;
    
    public NoMoneyState(VendingMachine machine) {
        this.machine = machine;
    }
    
    public void insertMoney() {
        System.out.println("Money inserted");
        machine.setState(machine.getHasMoneyState());
    }
    
    public void ejectMoney() {
        System.out.println("No money to eject");
    }
    
    public void dispense() {
        System.out.println("Insert money first");
    }
}

// Context
public class VendingMachine {
    private VendingMachineState noMoneyState;
    private VendingMachineState hasMoneyState;
    private VendingMachineState currentState;
    
    public VendingMachine() {
        noMoneyState = new NoMoneyState(this);
        hasMoneyState = new HasMoneyState(this);
        currentState = noMoneyState;
    }
    
    public void setState(VendingMachineState state) {
        this.currentState = state;
    }
    
    public void insertMoney() { currentState.insertMoney(); }
    public void ejectMoney() { currentState.ejectMoney(); }
    public void dispense() { currentState.dispense(); }
    
    public VendingMachineState getNoMoneyState() { return noMoneyState; }
    public VendingMachineState getHasMoneyState() { return hasMoneyState; }
}
```

---

### 20. Strategy Pattern
Family of algorithms, interchangeable.

```java
// Strategy
public interface PaymentStrategy {
    void pay(int amount);
}

// Concrete Strategies
public class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using Credit Card " + cardNumber);
    }
}

public class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    public void pay(int amount) {
        System.out.println("Paid " + amount + " using PayPal " + email);
    }
}

// Context
public class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(int amount) {
        paymentStrategy.pay(amount);
    }
}

// Usage
ShoppingCart cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardPayment("1234-5678"));
cart.checkout(100);

cart.setPaymentStrategy(new PayPalPayment("user@example.com"));
cart.checkout(50);
```

---

### 21. Template Method Pattern
Algorithm skeleton in base class.

```java
// Abstract Class with Template Method
public abstract class DataImporter {
    // Template method - defines the algorithm
    public final void importData(String filePath) {
        String data = readFile(filePath);
        validate(data);
        parse(data);
        save(data);
        notifyCompletion();
    }
    
    // Concrete methods
    protected String readFile(String filePath) {
        // Common implementation
        return "file content";
    }
    
    protected void notifyCompletion() {
        System.out.println("Import completed");
    }
    
    // Abstract methods - implemented by subclasses
    protected abstract void validate(String data);
    protected abstract void parse(String data);
    protected abstract void save(String data);
}

// Concrete Class
public class CsvImporter extends DataImporter {
    protected void validate(String data) {
        System.out.println("Validating CSV format");
    }
    
    protected void parse(String data) {
        System.out.println("Parsing CSV data");
    }
    
    protected void save(String data) {
        System.out.println("Saving CSV records");
    }
}

// Usage
DataImporter importer = new CsvImporter();
importer.importData("data.csv"); // Executes full workflow
```

---

### 22. Visitor Pattern
Separate operations from objects.

```java
// Element
public interface Shape {
    void accept(ShapeVisitor visitor);
}

// Concrete Elements
public class Circle implements Shape {
    private double radius;
    
    public void accept(ShapeVisitor visitor) {
        visitor.visit(this);
    }
    
    public double getRadius() { return radius; }
}

public class Rectangle implements Shape {
    private double width, height;
    
    public void accept(ShapeVisitor visitor) {
        visitor.visit(this);
    }
    
    public double getWidth() { return width; }
    public double getHeight() { return height; }
}

// Visitor
public interface ShapeVisitor {
    void visit(Circle circle);
    void visit(Rectangle rectangle);
}

// Concrete Visitor
public class AreaCalculator implements ShapeVisitor {
    private double totalArea;
    
    public void visit(Circle circle) {
        totalArea += Math.PI * circle.getRadius() * circle.getRadius();
    }
    
    public void visit(Rectangle rectangle) {
        totalArea += rectangle.getWidth() * rectangle.getHeight();
    }
    
    public double getTotalArea() { return totalArea; }
}

// Another Visitor
public class ShapeDrawer implements ShapeVisitor {
    public void visit(Circle circle) {
        System.out.println("Drawing circle");
    }
    
    public void visit(Rectangle rectangle) {
        System.out.println("Drawing rectangle");
    }
}

// Usage
List<Shape> shapes = Arrays.asList(new Circle(), new Rectangle());

AreaCalculator calculator = new AreaCalculator();
for (Shape shape : shapes) {
    shape.accept(calculator);
}
System.out.println("Total area: " + calculator.getTotalArea());

ShapeDrawer drawer = new ShapeDrawer();
for (Shape shape : shapes) {
    shape.accept(drawer);
}
```

---

## Quick Reference

### Pattern Categories
- **Creational**: Singleton, Factory, Abstract Factory, Builder, Prototype
- **Structural**: Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy
- **Behavioral**: Chain of Responsibility, Command, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor

### Modern Java Alternatives
- **Singleton**: Consider dependency injection (Spring) instead
- **Observer**: Use `PropertyChangeListener` or reactive streams (RxJava, Project Reactor)
- **Iterator**: Java's enhanced for-loop and Stream API
- **Strategy**: Java 8+ lambdas and method references
- **Template Method**: Functional interfaces and lambdas
- **Visitor**: Pattern matching (Java 16+ preview)

### When NOT to Use Patterns
- Simple problems (KISS principle)
- Premature abstraction
- When they add unnecessary complexity
- When language features suffice (lambdas, streams, etc.)
