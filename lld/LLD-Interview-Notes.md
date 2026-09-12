# Low-Level Design (LLD) — Interview Revision Notes

## 1. What is LLD?
LLD (Low-Level Design) describes how the system is broken into classes, interfaces, objects, responsibilities, relationships, and interactions.

**Interview line:**
> HLD focuses on components and architecture; LLD focuses on classes, objects, interfaces, responsibilities, and detailed interactions.

---

## 2. Core LLD Principles

### SOLID

#### S — Single Responsibility Principle
A class should have one reason to change.

**Memory:** One class → one responsibility.

#### O — Open/Closed Principle
Software entities should be open for extension but closed for modification.

**Memory:** Extend, don't keep modifying existing stable code.

#### L — Liskov Substitution Principle
A subtype should be usable wherever its base type is expected without breaking correctness.

**Memory:** Child should safely replace parent.

#### I — Interface Segregation Principle
Clients should not be forced to depend on methods they do not use.

**Memory:** Prefer small, focused interfaces.

#### D — Dependency Inversion Principle
High-level modules should depend on abstractions, not concrete implementations.

**Memory:** Depend on interface, not implementation.

---

## 3. DRY, KISS, YAGNI

### DRY
Don't Repeat Yourself — avoid duplicated business logic.

### KISS
Keep It Simple — prefer the simplest design that satisfies requirements.

### YAGNI
You Aren't Gonna Need It — don't add speculative functionality too early.

---

## 4. Cohesion vs Coupling

### High Cohesion
A class contains closely related responsibilities.

### Low Coupling
Classes have minimal unnecessary dependency on each other.

**Ideal:** High cohesion + Low coupling.

---

# 5. Composition vs Inheritance

### Inheritance
Represents **IS-A**.

```java
class Dog extends Animal {}
```

### Composition
Represents **HAS-A**.

```java
class Car {
    private Engine engine;
}
```

**Interview line:**
> I generally prefer composition over deep inheritance because composition makes dependencies explicit and reduces tight coupling.

---

# 6. Association / Aggregation / Composition

### Association
General relationship between objects.

### Aggregation
Weak whole-part relationship; parts can exist independently.

### Composition
Strong ownership; part's lifecycle is tied to the whole.

**Memory:**
- Association → uses/knows
- Aggregation → has, independent lifecycle
- Composition → owns, dependent lifecycle

---

# 7. Interface vs Abstract Class

### Interface
Used mainly for contracts/abstractions; a class can implement multiple interfaces.

### Abstract Class
Useful when related classes need shared state or implementation.

**Interview rule:**
> Use an interface when you need a contract/capability; use an abstract class when you need common state or shared base behavior.

---

# 8. Immutability
An immutable object cannot change state after creation.

Typical approach:
- final class where appropriate
- private final fields
- initialize through constructor
- no setters
- defensive copies for mutable fields

**Benefits:** easier reasoning, thread safety advantages, safer sharing.

---

# 9. Encapsulation
Keep object state private and expose controlled operations through methods.

```java
class BankAccount {
    private double balance;

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

---

# 10. Programming to an Interface
Prefer:

```java
PaymentService service = new StripePaymentService();
```

instead of tightly coupling callers to a concrete implementation.

This improves flexibility and testability.

---

# 11. Dependency Injection
Instead of creating dependencies inside a class, provide them from outside.

```java
class OrderService {
    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

**Benefits:** low coupling, easier testing, easier replacement of implementations.

---

# 12. Design Patterns — Creational

## Singleton ⭐⭐⭐
Ensures a class has one shared instance.

Common Java approach:

```java
public enum AppConfig {
    INSTANCE;
}
```

**Pitfalls:** global state, testing complexity, concurrency issues in incorrect implementations.

## Factory
Creates objects without exposing creation logic to the caller.

```java
interface Notification {}

class NotificationFactory {
    static Notification create(String type) {
        // choose implementation
        return null;
    }
}
```

**Memory:** Factory = object creation decision in one place.

## Abstract Factory
Creates related families of objects without specifying concrete classes.

## Builder ⭐⭐⭐
Useful for constructing complex objects step by step.

```java
User user = new User.Builder()
        .name("Nirbhay")
        .age(30)
        .build();
```

**Memory:** Builder = complex object, readable construction.

## Prototype
Creates objects by copying an existing object.

---

# 13. Structural Design Patterns

## Adapter ⭐⭐⭐
Converts one interface into another interface expected by the client.

**Memory:** Adapter = incompatible interfaces made compatible.

## Decorator ⭐⭐⭐
Adds behavior dynamically without changing the original class.

**Memory:** Decorator = wrap and add behavior.

## Facade ⭐⭐⭐
Provides a simple interface over a complex subsystem.

**Memory:** Facade = one simple entry point.

## Proxy ⭐⭐⭐
Provides a substitute/control layer around another object.

Common uses: access control, lazy loading, logging, remote calls.

## Composite
Treat individual objects and compositions uniformly using a tree structure.

## Flyweight ⭐⭐⭐
Shares reusable intrinsic state to reduce memory usage.

**Memory:** Flyweight = reuse objects, save memory.

---

# 14. Behavioral Design Patterns

## Strategy ⭐⭐⭐⭐⭐
Encapsulates interchangeable algorithms/behaviors.

```java
interface PaymentStrategy {
    void pay(double amount);
}
```

**Memory:** Strategy = choose behavior at runtime.

## Observer ⭐⭐⭐⭐
One-to-many dependency; observers are notified when subject state changes.

**Memory:** Subject changes → notify subscribers.

## Chain of Responsibility ⭐⭐⭐
Passes a request through a chain of handlers until one handles it or the chain ends.

**Memory:** Request travels through handlers.

## Template Method ⭐⭐⭐
Defines algorithm skeleton in a base class while allowing subclasses to customize certain steps.

## Command
Encapsulates a request as an object.

Useful for queues, undo/redo, scheduling.

## State
Object behavior changes based on its current state.

## Iterator
Provides sequential access without exposing underlying representation.

---

# 15. Factory vs Strategy vs Builder

### Factory
**Creates** an object.

### Strategy
**Chooses behavior/algorithm.**

### Builder
**Constructs a complex object step by step.**

🧠 **Create → Factory | Behave → Strategy | Build → Builder**

---

# 16. LLD Class Design Process ⭐⭐⭐⭐⭐

When given an LLD problem:

### Step 1 — Clarify requirements
Ask:
- What are the core use cases?
- What is in scope/out of scope?
- Single-user or multi-user?
- Concurrency needed?
- Persistence needed?
- Failure scenarios?

### Step 2 — Identify core entities
Example for parking lot:
- ParkingLot
- Floor
- ParkingSpot
- Vehicle
- Ticket
- Payment

### Step 3 — Assign responsibilities
Each class should have a clear responsibility.

### Step 4 — Define relationships
Use IS-A/HAS-A and composition where appropriate.

### Step 5 — Identify interfaces
Introduce abstractions where multiple implementations are expected.

### Step 6 — Select design patterns
Use patterns only where they solve a concrete design problem.

### Step 7 — Consider extensibility
Ask: what is likely to change?

### Step 8 — Discuss concurrency/error handling
Consider race conditions, locking, idempotency, retries, and validation when relevant.

---

# 17. Important LLD Questions

## Design Parking Lot
Typical entities:
- ParkingLot
- ParkingFloor
- ParkingSpot
- Vehicle
- Ticket
- EntryGate
- ExitGate
- Payment
- PricingStrategy

Useful patterns:
- Strategy → pricing/spot selection
- Factory → vehicle creation

## Design Elevator System
Entities:
- Elevator
- ElevatorController
- Request
- Floor

Discuss:
- scheduling strategy
- direction
- concurrency
- multiple elevators

## Design Tic-Tac-Toe
Entities:
- Board
- Player
- Piece
- Game

Use Strategy if choosing different winning/AI strategies.

## Design Snake and Ladder
Entities:
- Board
- Player
- Dice
- Snake
- Ladder
- Game

## Design Car Rental System
Entities:
- Vehicle
- Customer
- Reservation
- Branch
- Payment
- Rental

## Design Library Management System
Entities:
- Book
- BookItem
- Member
- Librarian
- Loan
- Fine

## Design ATM
Entities:
- ATM
- Card
- Account
- Transaction
- CashDispenser
- BankService

Use State pattern for ATM states where appropriate.

## Design Splitwise
Entities:
- User
- Expense
- Split
- Balance
- Group
- Settlement

Use Strategy for different split types.

## Design Notification System
Entities:
- Notification
- NotificationService
- EmailSender
- SmsSender
- PushSender

Use Strategy/Factory to support multiple channels.

## Design Rate Limiter
Entities:
- RateLimiter
- Rule
- Request

Algorithms:
- Fixed Window
- Sliding Window
- Token Bucket
- Leaky Bucket

---

# 18. Thread Safety in LLD

When multiple threads access shared mutable state:

Potential problems:
- race condition
- lost update
- inconsistent state

Possible solutions:
- synchronized
- ReentrantLock
- atomic classes
- immutable objects
- concurrent collections

**Interview line:**
> Before adding synchronization, I first identify the shared mutable state and the invariant that must remain consistent.

---

# 19. Idempotency
An operation is idempotent when repeating it produces the same intended final state.

Example:
- PUT is designed to be idempotent.
- A payment/command may need an idempotency key to avoid duplicate processing.

**LLD use:** payment, order creation, message processing.

---

# 20. SOLID Example — Payment System

Bad design:

```java
class PaymentService {
    void pay(String type) {
        if (type.equals("CARD")) {
            // card logic
        } else if (type.equals("UPI")) {
            // UPI logic
        }
    }
}
```

Better:

```java
interface PaymentMethod {
    void pay(double amount);
}

class CardPayment implements PaymentMethod {
    public void pay(double amount) {}
}

class UpiPayment implements PaymentMethod {
    public void pay(double amount) {}
}
```

Now new methods can be added with less impact on existing code.

---

# 21. Dependency Inversion in Spring
Spring's dependency injection is a practical example of programming against abstractions.

```java
@Service
class OrderService {
    private final PaymentService paymentService;

    OrderService(PaymentService paymentService) {
        this.paymentService = paymentService;
    }
}
```

---

# 22. Composition + Strategy Example

```java
class CheckoutService {
    private final PaymentStrategy strategy;

    CheckoutService(PaymentStrategy strategy) {
        this.strategy = strategy;
    }

    void checkout(double amount) {
        strategy.pay(amount);
    }
}
```

This avoids hard-coded `if/else` logic for every payment type.

---

# 23. Common LLD Follow-up Questions

### Why composition over inheritance?
> Composition reduces coupling and makes behavior replaceable.

### Why use interfaces?
> To define contracts and allow multiple implementations.

### When should I use a design pattern?
> When it solves a recurring design problem and improves clarity/extensibility; I avoid patterns just for the sake of using them.

### How do you make a class thread-safe?
> Minimize shared mutable state, prefer immutability where possible, and use appropriate synchronization/concurrency utilities where shared state is unavoidable.

### How do you make a design extensible?
> Identify likely variation points and isolate them behind abstractions such as interfaces/strategies.

### How do you improve testability?
> Use dependency injection, small responsibilities, interfaces where useful, and avoid hidden global/static state.

---

# 24. Pattern Recognition Cheat Sheet

| Problem | Pattern |
|---|---|
| Complex object construction | Builder |
| Object creation based on type | Factory |
| One shared instance | Singleton |
| Incompatible interfaces | Adapter |
| Add behavior by wrapping | Decorator |
| Simplify complex subsystem | Facade |
| Control/access around an object | Proxy |
| Share reusable objects | Flyweight |
| Interchangeable algorithms | Strategy |
| One-to-many notifications | Observer |
| Request through handlers | Chain of Responsibility |
| Algorithm skeleton | Template Method |
| Object changes behavior by state | State |
| Encapsulate request | Command |
| Tree hierarchy | Composite |

---

# 25. LLD vs HLD

| LLD | HLD |
|---|---|
| Classes/objects/interfaces | Services/components |
| Detailed interactions | Overall architecture |
| Design patterns | Distributed architecture |
| Method responsibilities | API/service boundaries |
| Object relationships | Databases, queues, caches, infrastructure |

**Memory:**
**HLD = Boxes**
**LLD = Classes inside the boxes**

---

# 26. Exceptional Interview Habits

1. Start with requirements before coding.
2. Keep classes focused.
3. Prefer interfaces for variable behavior.
4. Prefer composition over deep inheritance.
5. Make dependencies explicit through DI.
6. Mention thread safety when shared mutable state exists.
7. Discuss extensibility and likely change points.
8. Explain trade-offs instead of claiming there is only one design.
9. Use patterns only where justified.
10. Keep domain model separate from infrastructure details where practical.

---

# 27. 2-Minute LLD Answer Framework

When interviewer gives any LLD problem, say:

> **“First I will clarify the requirements and identify the core use cases. Then I will identify the main entities and assign a single responsibility to each class. I will define relationships using composition where appropriate and introduce interfaces for behavior that is likely to vary. Then I will select design patterns only where they simplify extensibility. Finally, I will discuss thread safety, error handling, persistence, and testability based on the requirements.”**

---

# 28. Ultimate Revision

### Remember:

**SOLID + DRY + KISS + YAGNI**

**IS-A → Inheritance**

**HAS-A → Composition**

**Create → Factory**

**Build → Builder**

**Choose behavior → Strategy**

**Notify many → Observer**

**Wrap behavior → Decorator**

**Convert interface → Adapter**

**Simplify subsystem → Facade**

**Control access → Proxy**

**Share objects → Flyweight**

**Request chain → Chain of Responsibility**

**State changes behavior → State**

**Algorithm skeleton → Template Method**

**Command as object → Command**

**High cohesion + Low coupling = Good LLD**
