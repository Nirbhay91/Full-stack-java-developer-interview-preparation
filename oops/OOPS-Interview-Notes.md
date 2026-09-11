# OOPS — Interview Notes

## 1. What is OOP?
Object-Oriented Programming organizes software around objects that contain state and behavior.

### Four pillars
- Encapsulation
- Inheritance
- Polymorphism
- Abstraction

## 2. Encapsulation
Wrapping data and methods together and controlling direct access to state, commonly using private fields with public/protected methods.

**Memory:** Encapsulation = hide data + controlled access.

## 3. Inheritance
A child class acquires accessible properties/behavior from a parent class using `extends`.

```java
class Animal {
    void eat() {}
}

class Dog extends Animal {
    void bark() {}
}
```

**Memory:** Inheritance = IS-A relationship.

## 4. Types of inheritance in Java
- Single
- Multilevel
- Hierarchical
- Multiple inheritance of classes is not supported.
- Multiple inheritance of type can be achieved through interfaces.

## 5. Polymorphism
One interface/reference can represent different implementations.

### Compile-time polymorphism
Method overloading.

### Runtime polymorphism
Method overriding / dynamic method dispatch.

```java
Notification n = new EmailNotify();
n.sendMessage();
```

The overridden method is selected based on the actual object at runtime.

**Memory:** Overriding -> Runtime -> Actual Object.

## 6. Abstraction
Showing essential behavior while hiding implementation details.

Achieved using:
- Abstract classes
- Interfaces

## 7. Abstract class vs interface
- Abstract class can have state, constructors, concrete methods and abstract methods.
- Interface defines a contract; Java 8 also allows default and static methods.
- A class can extend one class but implement multiple interfaces.

## 8. Overloading vs Overriding
| Overloading | Overriding |
|---|---|
| Same method name, different parameter list | Same signature in child class |
| Compile time | Runtime |
| Same class commonly | Parent-child relationship |
| Return type alone cannot overload | Return type may be covariant |

**Memory:** Overload = compile time; Override = runtime.

## 9. Constructor
Special member used to initialize an object.

- Same name as class
- No return type
- Not inherited
- Can be overloaded

## 10. `this` vs `super`
- `this` refers to current object.
- `super` refers to the parent-class portion / members of the parent.

## 11. `final`
- final variable -> cannot be reassigned
- final method -> cannot be overridden
- final class -> cannot be extended

## 12. `static`
Belongs to the class rather than individual objects.

## 13. Composition vs inheritance
Composition represents HAS-A and is often preferred when behavior should be assembled from collaborators instead of tightly coupling a class to a parent hierarchy.

```java
class Car {
    private Engine engine;
}
```

**Memory:**
- IS-A -> inheritance
- HAS-A -> composition

## 14. Why prefer composition over deep inheritance?
Deep inheritance can hide dependencies and make testing/maintenance harder. Composition makes dependencies explicit and easier to replace or mock.

## 15. Association / Aggregation / Composition
- Association -> general relationship
- Aggregation -> weak whole-part relationship; parts can exist independently
- Composition -> strong ownership; part lifetime is tied to the whole

## 16. Object class important methods
- `equals()`
- `hashCode()`
- `toString()`
- `getClass()`
- `clone()` (legacy-style API)

## 17. equals() and hashCode()
If two objects are equal according to `equals()`, they must have the same `hashCode()`.

When overriding `equals()`, override `hashCode()` consistently.

## 18. Immutable class
An immutable object's state cannot change after construction.

Common approach:
- final class
- private final fields
- initialize through constructor
- no setters
- defensive copies for mutable fields

`String` is a classic immutable type.

## 19. Shallow copy vs deep copy
- Shallow copy copies references to nested objects.
- Deep copy copies nested mutable state as well.

## 20. Upcasting / downcasting
```java
Animal a = new Dog(); // upcasting
Dog d = (Dog) a;      // downcasting
```
Downcasting should be done only when the runtime object is compatible; otherwise `ClassCastException` can occur.

## 21. IS-A vs HAS-A
- Dog IS-A Animal -> inheritance
- Car HAS-A Engine -> composition/aggregation

## 22. Interview rapid answers
**OOP pillars:** Encapsulation, Inheritance, Polymorphism, Abstraction.

**Runtime polymorphism:** Overridden method is selected based on the actual object type at runtime.

**Composition:** Prefer explicit object collaboration and lower coupling over deep inheritance when appropriate.

**Overloading:** Same method name with different parameter lists; resolved at compile time.

**Overriding:** Child provides its own implementation of an inherited method; resolved dynamically at runtime.
