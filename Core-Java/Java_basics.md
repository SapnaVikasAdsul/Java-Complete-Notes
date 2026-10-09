# Java Interview Prep Notes (Reviewed & Corrected)

> Cleaned up from your original notes. Fixed a few inaccurate statements (flagged with ⚠️ **Fixed** below) and filled in a couple of questions you'd left blank, marked ✅ **Added**. Everything else is your original content, just reformatted.

---

## 1. How a Java Program Runs

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello Java");
    }
}
```

**Execution flow:**

```
Java source (.java)
   ↓  javac (compiler)
Bytecode (.class)
   ↓
JVM loads the class
   ↓
Class Loader
   ↓
Bytecode Verifier
   ↓
JVM Execution Engine (interpreter + JIT compiler)
   ↓
Machine code
   ↓
Output
```

**Key point:** `javac` converts `.java` source into bytecode (`.class` file). The JVM then loads, verifies, and executes that bytecode — interpreting it and JIT-compiling "hot" code paths into native machine code for speed.

---

## 2. What Is Java, and Why Is It Still Popular?

Java is a platform and a language for building secure, scalable, high-performance applications. Its signature feature is **WORA — Write Once, Run Anywhere**: you compile once to bytecode, and any OS with a JVM can run it, without recompiling.

**Top 5 conceptual questions:**

1. **What is Java?** A high-level, object-oriented programming language used to build secure, platform-independent applications.
2. **Why is Java platform-independent?** Because source code compiles to bytecode, which any JVM (on any OS) can execute.
3. **What does "Write Once, Run Anywhere" mean?** Write and compile the code once; run the same bytecode on different operating systems without modification.
4. **Is Java compiled or interpreted?** Both. `javac` compiles source to bytecode; the JVM then interprets that bytecode and JIT-compiles frequently executed parts into native machine code.
5. **Why do enterprises prefer Java?** It's secure, reliable, scalable, has strong community/library support, and is well-suited to large enterprise systems.

---

## 3. OOP Concepts

Object-oriented programming (OOP) is a programming approach where software is designed using **classes** and **objects**.

- A **class** is a blueprint or template.
- An **object** is a real instance created from that blueprint.
- Objects have **state** (fields/variables) and **behavior** (methods).

**Real-life example:**  
A `Car` class is a blueprint. A specific car like "Honda City" or "Tesla Model 3" is an object. Each car has state such as color, speed, and fuel level, and behavior such as start, accelerate, and brake.

```java
class Car {
    String brand;
    String color;
    int speed;

    void start() {
        System.out.println(brand + " started");
    }

    void accelerate() {
        speed += 10;
        System.out.println("Speed: " + speed);
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.brand = "Honda";
        car.color = "White";
        car.start();
        car.accelerate();
    }
}
```

### Four Pillars of OOP

### 1. Encapsulation

**Definition:** Encapsulation means wrapping data and methods together inside a class and protecting the data from direct outside access. In Java, this is usually done by keeping variables `private` and exposing controlled access through getters and setters.

**Real-life example:**  
In an ATM, the user cannot directly access the bank's internal database. The user can only perform allowed operations such as withdraw, deposit, and check balance.

```java
class BankAccount {
    private double balance;

    public BankAccount(double balance) {
        this.balance = balance;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        BankAccount account = new BankAccount(1000);
        account.deposit(500);
        account.withdraw(300);
        System.out.println(account.getBalance());
    }
}
```

**Interview point:** Encapsulation improves security, data hiding, validation, and maintainability.

### 2. Abstraction

**Definition:** Abstraction means hiding implementation details and showing only the required functionality to the user. In Java, abstraction is achieved using **abstract classes** and **interfaces**.

**Real-life example:**  
When you drive a car, you use the steering, accelerator, and brake. You do not need to know the internal engine mechanism to drive it.

```java
interface Payment {
    void pay(double amount);
}

class CreditCardPayment implements Payment {
    public void pay(double amount) {
        System.out.println("Paid " + amount + " using credit card");
    }
}

class UpiPayment implements Payment {
    public void pay(double amount) {
        System.out.println("Paid " + amount + " using UPI");
    }
}

public class Main {
    public static void main(String[] args) {
        Payment payment = new UpiPayment();
        payment.pay(1500);
    }
}
```

**Interview point:** Abstraction reduces complexity and helps achieve loose coupling.

### 3. Inheritance

**Definition:** Inheritance allows one class to acquire the properties and methods of another class. The existing class is called the parent/superclass, and the new class is called the child/subclass.

**Real-life example:**  
A `Car` and a `Bike` are both types of `Vehicle`. Common behavior like `start()` can be defined once in `Vehicle` and reused by child classes.

```java
class Vehicle {
    void start() {
        System.out.println("Vehicle started");
    }
}

class Car extends Vehicle {
    void openBoot() {
        System.out.println("Car boot opened");
    }
}

public class Main {
    public static void main(String[] args) {
        Car car = new Car();
        car.start();
        car.openBoot();
    }
}
```

**Interview point:** Inheritance supports code reuse, but in real projects composition is often preferred when classes do not have a true "is-a" relationship.

### 4. Polymorphism

**Definition:** Polymorphism means "many forms". It allows the same method name or interface to behave differently based on the object or arguments.

**Types of polymorphism in Java:**
- **Compile-time polymorphism:** method overloading.
- **Runtime polymorphism:** method overriding.

**Real-life example:**  
The word "draw" can mean drawing a circle, rectangle, or triangle depending on the object.

**Compile-time polymorphism - method overloading:**

```java
class Calculator {
    int add(int a, int b) {
        return a + b;
    }

    double add(double a, double b) {
        return a + b;
    }
}
```

**Runtime polymorphism - method overriding:**

```java
class Animal {
    void sound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }
}

class Cat extends Animal {
    @Override
    void sound() {
        System.out.println("Cat meows");
    }
}

public class Main {
    public static void main(String[] args) {
        Animal animal = new Dog();
        animal.sound();

        animal = new Cat();
        animal.sound();
    }
}
```

**Interview point:** Polymorphism improves flexibility because the same reference type can point to different object implementations.

### Common Interview Questions

1. **What is the difference between class and object?**  
   A class is a blueprint; an object is an instance of that class.

2. **What is the difference between abstraction and encapsulation?**  
   Abstraction hides implementation details; encapsulation hides data by restricting direct access.

3. **What is the difference between overloading and overriding?**  
   Overloading happens in the same class with the same method name but different parameters. Overriding happens when a child class provides its own implementation of a parent class method.

4. **Can Java support multiple inheritance?**  
   Java does not support multiple inheritance with classes to avoid ambiguity, but it supports multiple inheritance of type through interfaces.

5. **Why is OOP useful?**  
   OOP improves code reusability, maintainability, readability, scalability, and testability.

### SOLID Principles

SOLID is a set of five object-oriented design principles. These principles help us write code that is easy to understand, maintain, test, and extend.

| Letter | Principle | Simple Meaning |
|---|---|---|
| S | Single Responsibility Principle | A class should have only one main responsibility. |
| O | Open/Closed Principle | Code should be open for extension but closed for modification. |
| L | Liskov Substitution Principle | A child class should be usable wherever its parent class is expected. |
| I | Interface Segregation Principle | Use small, specific interfaces instead of one large interface. |
| D | Dependency Inversion Principle | Depend on abstractions, not concrete classes. |

### 1. Single Responsibility Principle (SRP)

**Definition:** A class should have only one reason to change. It should focus on one responsibility.

**Real-life example:**  
In a restaurant, the chef cooks food and the cashier handles billing. If one person does both jobs, responsibilities become mixed and harder to manage.

**Bad design idea:**

```java
class EmployeeService {
    void saveEmployee() {
        // save employee to database
    }

    void sendEmail() {
        // send welcome email
    }

    void generateReport() {
        // generate employee report
    }
}
```

This class has multiple responsibilities: database logic, email logic, and report logic.

**Better design idea:**

```java
class EmployeeService {
    void saveEmployee() {
        // employee business logic
    }
}

class EmailService {
    void sendEmail() {
        // email logic
    }
}

class ReportService {
    void generateReport() {
        // report logic
    }
}
```

**Interview point:** SRP improves readability, testing, and maintainability.

### 2. Open/Closed Principle (OCP)

**Definition:** A class should be open for extension but closed for modification. This means we should be able to add new behavior without changing existing tested code.

**Real-life example:**  
A phone supports new apps without changing the phone's operating system code each time.

```java
interface PaymentMethod {
    void pay(double amount);
}

class CreditCardPayment implements PaymentMethod {
    public void pay(double amount) {
        System.out.println("Paid using credit card: " + amount);
    }
}

class UpiPayment implements PaymentMethod {
    public void pay(double amount) {
        System.out.println("Paid using UPI: " + amount);
    }
}

class CheckoutService {
    void checkout(PaymentMethod paymentMethod, double amount) {
        paymentMethod.pay(amount);
    }
}
```

If a new payment type is needed, create a new class like `NetBankingPayment`. No need to modify `CheckoutService`.

**Interview point:** OCP reduces the risk of breaking existing working code.

### 3. Liskov Substitution Principle (LSP)

**Definition:** A child class should be able to replace its parent class without breaking the application behavior.

**Real-life example:**  
If a payment machine accepts any card, then debit card and credit card should both work correctly wherever a card is expected.

**Bad design idea:**

```java
class Bird {
    void fly() {
        System.out.println("Bird is flying");
    }
}

class Penguin extends Bird {
    void fly() {
        throw new UnsupportedOperationException("Penguin cannot fly");
    }
}
```

This breaks LSP because `Penguin` cannot behave like a normal `Bird` if the parent says every bird can fly.

**Better design idea:**

```java
class Bird {
    void eat() {
        System.out.println("Bird is eating");
    }
}

interface Flyable {
    void fly();
}

class Sparrow extends Bird implements Flyable {
    public void fly() {
        System.out.println("Sparrow is flying");
    }
}

class Penguin extends Bird {
    // Penguin can eat, but does not need to fly
}
```

**Interview point:** LSP prevents incorrect inheritance.

### 4. Interface Segregation Principle (ISP)

**Definition:** A class should not be forced to implement methods it does not need. Prefer small, focused interfaces.

**Real-life example:**  
A basic printer should not be forced to support scan and fax features.

**Bad design idea:**

```java
interface Machine {
    void print();
    void scan();
    void fax();
}
```

If a simple printer implements this interface, it may be forced to provide empty or unsupported methods.

**Better design idea:**

```java
interface Printer {
    void print();
}

interface Scanner {
    void scan();
}

class BasicPrinter implements Printer {
    public void print() {
        System.out.println("Printing document");
    }
}

class MultiFunctionPrinter implements Printer, Scanner {
    public void print() {
        System.out.println("Printing document");
    }

    public void scan() {
        System.out.println("Scanning document");
    }
}
```

**Interview point:** ISP keeps interfaces clean and avoids unnecessary method implementation.

### 5. Dependency Inversion Principle (DIP)

**Definition:** High-level modules should not directly depend on low-level modules. Both should depend on abstractions such as interfaces.

**Real-life example:**  
A remote control depends on a general TV interface, not on the internal wiring of a specific TV brand.

**Bad design idea:**

```java
class MySqlDatabase {
    void save() {
        System.out.println("Saving to MySQL");
    }
}

class UserService {
    private MySqlDatabase database = new MySqlDatabase();

    void saveUser() {
        database.save();
    }
}
```

Here, `UserService` is tightly coupled to `MySqlDatabase`.

**Better design idea:**

```java
interface Database {
    void save();
}

class MySqlDatabase implements Database {
    public void save() {
        System.out.println("Saving to MySQL");
    }
}

class OracleDatabase implements Database {
    public void save() {
        System.out.println("Saving to Oracle");
    }
}

class UserService {
    private Database database;

    UserService(Database database) {
        this.database = database;
    }

    void saveUser() {
        database.save();
    }
}
```

Now `UserService` depends on the `Database` abstraction, so we can pass `MySqlDatabase`, `OracleDatabase`, or any other implementation.

**Interview point:** DIP helps achieve loose coupling and is the base idea behind dependency injection in Spring.

**Simple interview answer:** SOLID principles help developers design clean, loosely coupled, testable, and maintainable object-oriented code.

---

## 4. JVM Memory Areas ✅ *Added*

| **Heap** | Stores all objects and instance variables; shared across threads; managed by GC. |
| **Stack** | One per thread; stores method call frames, local variables, and partial results. |
| **Method Area / Metaspace** | Stores class metadata, static variables, constant pool (Metaspace replaced PermGen from Java 8 onward). |
| **PC (Program Counter) Register** | Per thread; tracks the address of the currently executing instruction. |
| **Native Method Stack** | Supports native (non-Java, e.g. JNI/C) method calls. |

### Garbage Collection
GC reclaims memory used by unreachable objects.
- **Minor GC** — cleans the young generation (short-lived objects); fast and frequent.
- **Major/Full GC** — cleans the old generation (or the whole heap); more expensive and can cause longer pauses.

---

## 5. `final` vs `finally` vs `finalize`

| Keyword | Meaning |
|---|---|
| `final` | Restricts modification — a final variable can't be reassigned, a final method can't be overridden, a final class can't be extended. |
| `finally` | A block in exception handling that always executes after `try`/`catch`, whether or not an exception occurred — used for cleanup (closing DB connections, files). |
| `finalize()` | A method on `Object`, historically called by the GC before an object is destroyed. **Deprecated since Java 9** because its execution timing is unpredictable and it hurts performance — don't rely on it. |

---

## 6. Why Can't a Static Method Access Non-Static Variables?

A static method belongs to the **class**, not to any object, and static members are set up as soon as the JVM loads the class into memory. Non-static (instance) variables don't exist until an object is created — so at the point a static method could run, instance variables may not exist yet, and the compiler disallows the reference.

---

## 7. Java 8 / 17 / 21 — New Features

### Java 8

**1. Lambda Expressions**

```java
// Before Java 8
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// Java 8
Runnable r = () -> System.out.println("Hello");
```
Less code, more readable, and used heavily with the Stream API.

**2. Functional Interfaces** — an interface with exactly one abstract method.

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}

Calculator cal = (a, b) -> a + b;
System.out.println(cal.add(10, 20));
```
Common built-in ones: `Predicate<T>`, `Function<T,R>`, `Consumer<T>`, `Supplier<T>`.

**3. Stream API** — performs bulk operations on collections (filter, transform, sort, aggregate), reducing code length.

```java
// Without streams
for (String name : names) {
    if (name.startsWith("A")) System.out.println(name);
}

// With streams
names.stream()
     .filter(name -> name.startsWith("A"))
     .forEach(System.out::println);
```
Common operations: `filter()`, `map()`, `sorted()`, `distinct()`, `limit()`, `collect()`, `count()`, `findFirst()`.

**4. Method References**

```java
names.forEach(System.out::println);
```
Types: static method reference, instance method reference, constructor reference.

**5. Default Methods** — interfaces can now provide implementations, so adding a method no longer breaks existing implementers.

```java
interface Vehicle {
    default void start() {
        System.out.println("Vehicle Started");
    }
}
```

**6. Static Methods in Interfaces**

```java
interface MathUtil {
    static int square(int x) { return x * x; }
}
MathUtil.square(5);
```

**7. New Date & Time API (`java.time`)** — immutable, thread-safe, and easier to use than `Date`/`Calendar`.

```java
LocalDate date = LocalDate.now();
LocalTime time = LocalTime.now();
LocalDateTime dateTime = LocalDateTime.now();
```

**8. `Iterable.forEach()`**

```java
names.forEach(System.out::println);
```

### Java 17 (LTS)

**1. Records** — eliminate boilerplate for simple data carriers.

```java
public record Employee(int id, String name) {}

Employee emp = new Employee(1, "John");
System.out.println(emp.name());
```
Java auto-generates the constructor, accessors (`id()`, `name()`), `equals()`, `hashCode()`, and `toString()`.

**2. Sealed Classes** — restrict which classes/interfaces may extend or implement a type, giving explicit control over the inheritance hierarchy.

```java
public abstract sealed class Shape permits Circle, Rectangle {}
public final class Circle extends Shape { /* ... */ }
public final class Rectangle extends Shape { /* ... */ }
```
Useful for domain modeling and enables exhaustiveness checks in `switch`.

**3. Pattern Matching for `switch` (Preview in 17)**

```java
public String formatShape(Shape shape) {
    return switch (shape) {
        case Circle c -> "Circle with radius " + c.radius();
        case Rectangle r -> "Rectangle with width " + r.width() + " and height " + r.height();
        default -> "Unknown shape";
    };
}
```

**4. Performance improvements** — better GC, faster startup, better memory usage, JIT optimizations — often "free" speedups with no code changes.

### Java 21 (LTS)

**1. Pattern Matching for `switch`** — now a **finalized**, standard feature (same syntax as above).

**2. Virtual Threads (finalized)** — lightweight threads managed by the JVM, ideal for I/O-heavy, high-concurrency workloads (e.g., a Spring Boot app handling thousands of requests that spend most of their time waiting on DB/HTTP calls).

```java
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    executor.submit(() -> System.out.println("Running in a virtual thread!"));
}
```
They make the classic thread-per-request model scale much further, since a blocked virtual thread doesn't tie up an OS thread.

**3. Sequenced Collections** — a new interface for collections with a well-defined encounter order (first/last access, reversed view).

```java
SequencedCollection<String> collection = new LinkedHashSet<>();
collection.add("one");
collection.add("two");
collection.add("three");
collection.forEach(System.out::println);
```

### Summary Table

| Java 8 | Java 17 | Java 21 |
|---|---|---|
| Lambda Expressions | Records | Virtual Threads (final) |
| Stream API | Sealed Classes | Record Patterns |
| Functional Interfaces | Pattern Matching for `instanceof` | Pattern Matching for `switch` (final) |
| `Optional` | Text Blocks | Sequenced Collections |
| `java.time` API | Switch Expressions | Generational ZGC |
| Default Interface Methods | Strong Encapsulation (JDK internals) | Additional preview language features |

---

## 8. Comparable vs Comparator ✅ *Added*

| | Comparable | Comparator |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `compareTo(T o)` | `compare(T o1, T o2)` |
| Where defined | Inside the class being compared (`implements Comparable<T>`) | A separate class/lambda, external to the object |
| Ordering | Defines a single, "natural" ordering | Lets you define multiple, custom orderings |
| Use | `Collections.sort(list)` | `Collections.sort(list, comparator)` |

Rule of thumb: use `Comparable` for a class's one true default ordering; use `Comparator` when you need several different sort orders (e.g., by name, then separately by age) without modifying the class itself.

---

## 9. Why Passwords Should Be Stored in `char[]`, Not `String`

```java
String password = "MySecret123";   // avoid this
```
`String` is immutable — once created, you can't modify its contents. Even setting `password = null` only removes your reference; the original characters may linger in heap memory until garbage collected.

```java
char[] password = {'M','y','S','e','c','r','e','t'};
// after use:
Arrays.fill(password, '\0');
```
`char[]` is mutable, so you can explicitly overwrite it right after use, minimizing how long the sensitive data stays readable in memory.

`ArrayList<Character>` isn't ideal either — it wraps boxed `Character` objects rather than primitive `char` values, adding object/reference overhead and making it harder to guarantee every copy is cleared.

---

## 10. Why Java Doesn't Allow Generic Array Creation

```java
T[] arr = new T[10];              // compile-time error
new List<String>[10];             // compile-time error

String[] arr = new String[10];    // fine — concrete type known at runtime
```

| Arrays | Generics |
|---|---|
| Know their element type at runtime | Type is erased after compilation (type erasure) |

Since generic type information doesn't exist at runtime, the JVM can't enforce type-safety on a generic array, so Java disallows creating one directly.

---

## 11. Why Is `String` Immutable in Java?

- **Security** — sensitive values (passwords, API keys, DB connection strings) are often held as strings; immutability prevents them from being altered mid-flight, which would be a security risk.
- **Performance (String Pool)** — identical string literals share the same object in the string pool rather than each allocating a new object, saving memory.
- **HashMap correctness** — a string's hash code is computed once and cached. If strings were mutable, a key's hash code could change after insertion, and it would become unfindable in a hash-based collection.
- **Thread safety** — immutable objects can be shared across threads without synchronization.

---

## 12. `==` vs `equals()` vs `hashCode()`

These three are commonly asked together because they are used when comparing objects and when storing objects in hash-based collections like `HashMap` and `HashSet`.

| Concept | Simple Meaning |
|---|---|
| `==` | Compares primitive values directly; for objects, compares whether both references point to the same object in memory. |
| `equals()` | Compares the logical content/value of objects, if the class has overridden it properly. |
| `hashCode()` | Returns an integer hash value used by hash-based collections to quickly find where an object may be stored. |

### 1. `==`

**Definition:** `==` checks whether two values are exactly the same. For primitives, it compares values. For objects, it compares references.

```java
int a = 10;
int b = 10;
System.out.println(a == b); // true

String s1 = new String("Java");
String s2 = new String("Java");
System.out.println(s1 == s2); // false, different objects
```

**Real-life example:**  
Two people may have the same name, but they are not the same person. `==` checks whether both references point to the exact same person/object.

### 2. `equals()`

**Definition:** `equals()` is used to compare the meaningful content of objects. By default, `Object.equals()` behaves like `==`, but classes such as `String`, wrapper classes, and custom classes can override it.

```java
String s1 = new String("Java");
String s2 = new String("Java");

System.out.println(s1 == s2);      // false
System.out.println(s1.equals(s2)); // true
```

**Real-life example:**  
Two ID cards may be physically different cards, but if both contain the same employee ID, they represent the same employee logically.

### 3. `hashCode()`

**Definition:** `hashCode()` returns an integer hash value for an object. Collections like `HashMap` and `HashSet` use it to decide where to store and search for objects.

**Important rule:**
- If two objects are equal using `equals()`, they must have the same `hashCode()`.
- If two objects have the same `hashCode()`, they may or may not be equal because hash collisions are possible.

```java
import java.util.Objects;

class Employee {
    private int id;
    private String name;

    Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) {
            return true;
        }

        if (obj == null || getClass() != obj.getClass()) {
            return false;
        }

        Employee employee = (Employee) obj;
        return id == employee.id && Objects.equals(name, employee.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}

public class Main {
    public static void main(String[] args) {
        Employee e1 = new Employee(101, "Ravi");
        Employee e2 = new Employee(101, "Ravi");

        System.out.println(e1 == e2);             // false
        System.out.println(e1.equals(e2));        // true
        System.out.println(e1.hashCode() == e2.hashCode()); // true
    }
}
```

### How `HashMap` Uses Them

When you put a key in a `HashMap`:

1. Java calls `hashCode()` to find the bucket location.
2. If multiple keys are in the same bucket, Java calls `equals()` to find the exact matching key.

```java
Map<Employee, String> map = new HashMap<>();
map.put(new Employee(101, "Ravi"), "Developer");

System.out.println(map.get(new Employee(101, "Ravi"))); // Developer
```

This works correctly only because both `equals()` and `hashCode()` are overridden.

**Interview point:** Whenever you override `equals()`, you should also override `hashCode()`. Otherwise, hash-based collections like `HashMap`, `HashSet`, and `Hashtable` may behave incorrectly.

---

## 13. Functional Interfaces Used in the Collections Framework

| Functional Interface | Method | Signature | Common Collection Usage |
|---|---|---|---|
| `Predicate<T>` | `test()` | `T → boolean` | `filter()`, `removeIf()` |
| `Consumer<T>` | `accept()` | `T → void` | `forEach()` |
| `Function<T,R>` | `apply()` | `T → R` | `map()` |
| `Supplier<T>` | `get()` | `() → T` | Creating/providing values |
| `Comparator<T>` | `compare()` | `T,T → int` | `sort()` |
| `BiConsumer<T,U>` | `accept()` | `T,U → void` | `Map.forEach()` |
| `UnaryOperator<T>` | `apply()` | `T → T` | `replaceAll()` |

---

## 14. Runnable vs Callable

| Feature | `Runnable` | `Callable<V>` |
|---|---|---|
| Package | `java.lang` | `java.util.concurrent` |
| Method | `run()` | `call()` |
| Return value | None | Returns a value (`V`) |
| Checked exceptions | Cannot declare/throw | Can throw checked exceptions |
| Functional interface | Yes | Yes |
| Typical use | Fire-and-forget task | Task where a result is needed |
| Result via `ExecutorService` | `Future<?>` | `Future<T>` |

```java
Runnable task = () -> System.out.println("Processing customer...");
Thread thread = new Thread(task);
thread.start();
```

```java
Callable<String> task = () -> "Customer created successfully";
ExecutorService executor = Executors.newFixedThreadPool(2);
Future<String> future = executor.submit(task);
System.out.println(future.get());
executor.shutdown();
```

---

## 15. Lambda Expressions & Functional Interfaces

A lambda expression has no name, no modifiers, and no explicit return type:

```java
(a, b) -> { return a + b; }
```

Benefits: fewer lines of code, supports sequential/parallel execution by passing behavior as an argument, cleaner API-call code, more readable and maintainable.

A **functional interface** has exactly one abstract method (e.g., `Runnable`, `Comparable`, `Comparator`, `Callable`). Lambdas need a target functional interface to be assigned to.

**Three ways to implement a functional interface:**

```java
// 1. Separate class implementing the interface
class MyRunnable implements Runnable {
    public void run() { System.out.println("separate class"); }
}

// 2. Anonymous class
Runnable r = new Runnable() {
    @Override
    public void run() { System.out.println("anonymous class"); }
};
r.run();

// 3. Lambda expression
Runnable t1 = () -> System.out.println("using lambda");
new Thread(t1).start();
```

### Adapting one functional interface to another ✅ *Added*
Given:
```java
interface A { int method(int a); }
interface B { int method(int a, int b); }
```
You can't directly assign a lambda meant for `B` to `A` (different arity), but you can adapt one to the other by fixing one parameter — e.g., implement `A` in terms of a `B` instance with a captured second value:

```java
B b = (x, y) -> x + y;
int fixedValue = 10;
A a = x -> b.method(x, fixedValue);   // adapts B into A by capturing one argument

System.out.println(a.method(5));  // 15
```

### Method References ✅ *Added*
A method reference is shorthand for a lambda that just calls an existing method.

```java
// Static method reference
Function<String, Integer> parse = Integer::parseInt;

// Instance method reference on a particular object
String str = "hello";
Supplier<Integer> len = str::length;

// Instance method reference on an arbitrary object of a type
Function<String, Integer> len2 = String::length;

// Constructor reference
Supplier<ArrayList<String>> listMaker = ArrayList::new;
```

---

## 16. Collections Framework

**Arrays vs Collections**

| Arrays | Collections |
|---|---|
| Good raw performance | Generally slower than arrays |
| Fixed size | Growable |
| Homogeneous elements only | Can hold homogeneous or heterogeneous objects |
| No built-in algorithms | Built on standard data structures, so ready-made methods are available |

A **Collection** is a group of individual objects. The **Collection Framework** is a set of interfaces and classes for storing and manipulating groups of objects. Every collection interface extends `Serializable`. Key interfaces include `Collection`, `List`, `Set`, `SortedSet`, `Queue`, and `Map` (note: `Map` is *not* a subtype of `Collection`).

**`Collection` interface — common methods:** `add(Object o)`, `addAll(Collection c)`, `remove(Object o)`, `removeAll(Collection c)`, `clear()`, `isEmpty()`, `size()`, `contains(Object o)`, `containsAll(Collection c)`, `iterator()`.

> ⚠️ **Fixed:** `containsAll` takes a `Collection`, not a single `Object` — i.e. `containsAll(Collection c)`.

`Collection` (interface) vs `Collections` (utility class with static helper methods like `sort`, `unmodifiableList`, etc.) are different things.

### List Interface
Duplicates allowed, insertion order preserved. Methods: `add(int index, Object o)`, `addAll(int index, Collection c)`, `remove(int i)`, `indexOf(Object o)`, `get(int i)`, `set(int i, Object o)` (replaces).

- **`ArrayList`** — resizable array, duplicates allowed, insertion order preserved, heterogeneous elements allowed, `null` allowed. Insertion/deletion in the middle requires shifting elements (not ideal for that), but implements `RandomAccess` so retrieval by index is fast.

  `new ArrayList()` creates an empty list with a default initial capacity of 10.

  > ⚠️ **Fixed:** Growth formula was listed as `capacity*3/2 + 1`. The actual JDK formula (in `ArrayList.grow()`) is:
  > ```
  > newCapacity = oldCapacity + (oldCapacity >> 1)   // i.e., 1.5× the old capacity, no "+1"
  > ```

- **`LinkedList`** — doubly linked list, insertion order preserved, duplicates allowed, heterogeneous, `null` allowed. Best for insertion/deletion in the middle (no shifting), weaker for random-access retrieval than `ArrayList`.
- **`Vector`** — legacy, synchronized (thread-safe), otherwise similar to `ArrayList`.
- **`Stack`** — extends `Vector`, LIFO (last-in, first-out) structure.

### Set Interface
No duplicates; insertion order not guaranteed.

- **`HashSet`** — backed by a hash table, no duplicates, order not preserved (based on hash code), heterogeneous objects and `null` allowed, fast for search.
- **`LinkedHashSet`** — hash table + linked list, preserves insertion order, good for cache-like use cases.
- **`TreeSet`** — backed by a balanced (red-black) tree, no duplicates, sorted order (not insertion order), heterogeneous objects not allowed (they must be mutually comparable). `null` is only insertable as the very first element into an *empty* `TreeSet` (before any comparison is needed) — any further insertion attempt involving `null` throws `NullPointerException`.

| | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| Order | No order | Insertion order | Sorted order |
| Duplicates | Not allowed | Not allowed | Not allowed |
| Null elements | One, order n/a | One | Only as sole element of an empty set |
| Underlying DS | Hash table | Hash table + doubly linked list | Red-black tree |
| Performance | Fast | Fast | Slower |
| Use case | Order doesn't matter | Insertion order matters | Sorted order needed |

**List vs Set**

| List | Set |
|---|---|
| Duplicates allowed | Duplicates not allowed |
| Insertion order preserved | Not guaranteed (except `LinkedHashSet`) |

### SortedSet Interface
Duplicates not allowed; elements must be inserted according to some sorting order.

### Queue Interface
FIFO-style structure (details not expanded in the original notes — worth a follow-up read on `PriorityQueue`, `ArrayDeque`).

### Map Interface
**Not** part of the `Collection` hierarchy. Stores data as key-value pairs; duplicate keys not allowed. Implementations: `HashMap`, `LinkedHashMap`, `TreeMap`, `Hashtable` (legacy).

**Important interview correction:** `HashMap` is part of the **Java Collections Framework**, but it is not part of the **Collection interface hierarchy**.

Why?
- `Collection` represents a group of individual elements, such as `List`, `Set`, and `Queue`.
- `Map` represents key-value pairs, not single elements.
- Because of this, `Map` does not extend `Collection`, and `HashMap` implements `Map`, not `Collection`.

```java
Collection<String> names = new ArrayList<>(); // stores individual values

Map<Integer, String> employees = new HashMap<>(); // stores key-value pairs
employees.put(101, "Ravi");
```

**Simple example:**  
A `List` stores names directly: `["Ravi", "Amit"]`.  
A `HashMap` stores data using a key: `{101="Ravi", 102="Amit"}`.

**Cursors:** `Enumeration`, `Iterator`, `ListIterator`.

**Utility classes:** `Collections`, `Arrays`.

### HashMap vs Hashtable vs TreeMap vs LinkedHashMap

| Feature | HashMap | Hashtable | TreeMap | LinkedHashMap |
|---|---|---|---|---|
| Structure | Key-value pairs | Key-value pairs | Sorted key-value pairs | Key-value pairs |
| Thread-safe | No | Yes | No | No |
| Synchronized | No | Yes | No | No |
| Performance | Generally faster | Slower (sync overhead) | Slower | Fast |
| Null keys/values | 1 null key, multiple null values | No null keys or values | No null keys, null values allowed | 1 null key, multiple null values |
| Ordering | No guaranteed order | No guaranteed order | Sorted by key | Insertion order maintained |
| Duplicate keys | Not allowed (values can repeat) | Same | Same | Same |

Because `Hashtable` is a legacy class and its synchronization creates contention, modern code generally uses `HashMap` for non-concurrent scenarios and `ConcurrentHashMap` when concurrent access is required.

```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "Rahul");
map.put(2, "Amit");
System.out.println(map.get(1));
```

### Internal Structure of `HashMap`

`HashMap` stores data as key-value pairs. Internally, it uses an array of buckets. Each bucket can store one or more nodes.

Each node contains:

```java
hash
key
value
next
```

Think of a `HashMap` like a locker room:
- `hashCode()` helps decide which locker/bucket should be used.
- `equals()` helps find the exact key inside that bucket.
- The value is stored with the matching key.

### How `put()` Works

For `map.put("name", "Ravi")`:

1. Java calls `"name".hashCode()`.
2. HashMap calculates the bucket index using the hash.
3. If the bucket is empty, it stores the new node there.
4. If the bucket already has nodes, it checks keys using `equals()`.
5. If the same key already exists, the old value is replaced.
6. If the key is new, the node is added to that bucket.

```java
Map<String, String> map = new HashMap<>();
map.put("name", "Ravi");
map.put("city", "Pune");
map.put("name", "Amit"); // replaces Ravi because key "name" already exists

System.out.println(map.get("name")); // Amit
```

### How `get()` Works

For `map.get("name")`:

1. Java calls `"name".hashCode()`.
2. HashMap calculates the same bucket index.
3. It goes to that bucket.
4. It compares keys using `equals()`.
5. If the key matches, it returns the value.

### Collision in `HashMap`

A collision happens when two different keys produce the same bucket index.

```java
map.put("Aa", "First");
map.put("BB", "Second");
```

If both keys go to the same bucket, HashMap stores them in that bucket and uses `equals()` to identify the correct key.

In Java 8 and later:
- Small bucket collisions are stored as a linked list.
- If too many nodes are in one bucket, the linked list can become a balanced tree.
- This improves search performance inside that bucket.

### Resizing / Rehashing

By default, `HashMap` starts with capacity `16` and load factor `0.75`.

That means resizing happens when the map becomes more than 75% full.

```java
threshold = capacity * loadFactor
threshold = 16 * 0.75 = 12
```

When the 13th entry is added:

1. HashMap creates a bigger internal array.
2. Existing entries are redistributed into new buckets.
3. This process is called resizing or rehashing.

### Performance

| Operation | Average Case | Worst Case |
|---|---|---|
| `put()` | O(1) | O(n), or O(log n) when bucket is treeified |
| `get()` | O(1) | O(n), or O(log n) when bucket is treeified |
| `remove()` | O(1) | O(n), or O(log n) when bucket is treeified |

**Interview point:** A good `hashCode()` spreads keys evenly across buckets. If many keys go into the same bucket, performance decreases.

---

## 17. Project & Deployment Experience Notes (Personal — for interview storytelling)

### Deployment Process
Application deployed on AWS; deployment infrastructure/releases managed by the DevOps team, backend development owned by you.

**Flow:** commit → PR → code review/approval → merge → CI/CD builds the Spring Boot app with Maven, runs tests, produces a JAR or Docker image → deployed to the target AWS environment (dev → QA/UAT → production, validated at each stage) → smoke testing of key REST APIs → log/error monitoring for stability. On production issues: check logs and recent deployment changes, coordinate with DevOps, and roll back if needed.

### Project 1 — Customer Management Application
Layered architecture: Controller (REST) → Service (business logic) → Repository (Spring Data JPA) → Entity (mapped to Oracle tables), with DTOs separating API models from persistence models.

Your role: built create/retrieve/delete customer REST APIs, validations, duplicate-email checks, exception handling, correct HTTP status codes; configured the Oracle connection via JPA/Hibernate.

Stack: Java, Spring Boot, Spring Web, Spring Data JPA, Hibernate, Oracle, Maven, Postman, Git.

**Challenge — duplicate customers:** duplicate emails initially caused raw DB constraint errors (generic 500s). Fixed with service-level validation plus a unique DB constraint, returning a proper `409 Conflict` instead.

**Challenge — Oracle integration:** getting the service name, sequence, and entity mapping right; resolved by validating the connection in SQL Developer and aligning JPA mappings with the schema.

### Project 2 — Bulk Allocation Micro-Frontend
React 18 micro-frontend integrated into an Angular 17 host shell via Webpack 5 Module Federation. The React remote exposed itself through `remoteEntry.js`; the Angular host loaded it dynamically via a wrapper component using `mount()`/`unmount()` lifecycle methods.

Your role: built the React micro-frontend, reusable components, state via Redux Toolkit, routing, API integration, and the Module Federation setup.

Stack: React 18, Redux Toolkit, React Router, Tailwind CSS, Webpack 5 Module Federation, Git, GitHub, Angular 17 (host).

**Challenges:** Angular–React interoperability, lifecycle management, shared dependency/version conflicts, routing integration, and ensuring the micro-frontend could deploy independently without breaking the host.

---

## 18. Production/System-Design Questions to Prepare (left as practice prompts)

These were listed in your notes without answers yet — good candidates for a dedicated deep-dive session:

1. Your API suddenly becomes slow in production — how do you find the bottleneck?
2. Why can a `HashMap` cause problems when accessed by multiple threads?
3. Spring Boot API has high latency but low CPU usage — what do you investigate?
4. How would you debug an `OutOfMemoryError` in production?
5. How would you make a payment API idempotent?
6. A downstream service becomes slow — how do you prevent cascading failures?
7. Kafka consumer lag keeps increasing — what do you check?
8. Two requests update the same record simultaneously — how do you handle it?
9. App works fine with 100 users but fails at 5,000 — how do you debug it?
10. A DB query is fast locally but slow in production — why could that happen?
11. How would you design retries without causing a retry storm?
12. How would you trace latency across multiple microservices?
13. How would you handle repeatedly failing Kafka messages?
14. How would you prevent microservices from becoming a "distributed monolith"?

*Also worth answering for yourself before an interview: "Which Java version does your current project use, and why?" — this is project-specific, so plug in your actual answer (e.g., version + LTS support, team standard, library compatibility).*
