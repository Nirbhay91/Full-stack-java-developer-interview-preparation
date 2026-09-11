# Java 8 Interview Preparation

## 1. Main Features

Memory trick: **L-F-S-O-M-D-C**

- **L**ambda Expressions
- **F**unctional Interfaces
- **S**tream API
- **O**ptional
- **M**ethod References
- **D**efault and Static methods in interfaces
- **C**ollectors + Date/Time API

---

## 2. Lambda Expression

**Interview answer:** Lambda expression is a concise way of providing an implementation of a functional interface.

```java
Runnable r = () -> System.out.println("Hello");
```

Memory trick: **Lambda = short implementation**

---

## 3. Functional Interface

A functional interface has **exactly one abstract method**.

```java
@FunctionalInterface
interface Calculator {
    int add(int a, int b);
}

Calculator c = (a, b) -> a + b;
```

It may also contain multiple default and static methods.

Common interfaces:
- Runnable
- Comparator
- Predicate
- Function
- Consumer
- Supplier

Memory trick: **Functional = one abstract method**

---

## 4. Predicate / Function / Consumer / Supplier

```text
Predicate  -> input -> boolean       -> Test
Function   -> input -> output        -> Transform
Consumer   -> input -> no output     -> Consume
Supplier   -> no input -> output     -> Supply
```

Examples:

```java
Predicate<Integer> p = x -> x > 10;
Function<Integer, Integer> f = x -> x * 2;
Consumer<String> c = x -> System.out.println(x);
Supplier<String> s = () -> "Hello";
```

Also remember `BiPredicate`, `BiFunction`, and `BiConsumer` for two inputs.

---

## 5. Stream API

**Interview answer:** Stream API is used to process data from collections and other sources in a declarative and functional style.

A stream does not store data; it processes data.

### Stream pipeline

**Source -> Intermediate operations -> Terminal operation**

```java
List<Integer> result = numbers.stream()
        .filter(x -> x > 10)
        .map(x -> x * 2)
        .collect(Collectors.toList());
```

Memory trick: **S-I-T**

### Intermediate operations

- filter()
- map()
- flatMap()
- distinct()
- sorted()
- limit()
- skip()
- peek()

Intermediate operations are generally **lazy**.

### Terminal operations

- collect()
- forEach()
- reduce()
- count()
- min()
- max()
- findFirst()
- findAny()
- anyMatch()
- allMatch()
- noneMatch()

---

## 6. filter() vs map() vs flatMap()

### filter

Select/remove elements according to a condition.

```java
numbers.stream()
       .filter(x -> x % 2 == 0)
       .collect(Collectors.toList());
```

### map

Transforms each element.

```java
numbers.stream()
       .map(x -> x * 2)
       .collect(Collectors.toList());
```

### flatMap

Transforms and flattens nested structures.

```java
List<String> result = lists.stream()
        .flatMap(List::stream)
        .collect(Collectors.toList());
```

Memory trick: **filter = select, map = change, flatMap = flatten**

---

## 7. reduce()

Combines many elements into one result.

```java
int sum = numbers.stream()
        .reduce(0, (a, b) -> a + b);
```

Memory trick: **Many -> One**

---

## 8. findFirst() vs findAny()

- `findFirst()` returns the first element according to encounter order when an order exists.
- `findAny()` may return any matching element and can be useful with parallel streams.
- Both return `Optional<T>`.

---

## 9. anyMatch / allMatch / noneMatch

```text
anyMatch  -> at least one
allMatch  -> all
noneMatch -> none
```

```java
boolean exists = list.stream().anyMatch(x -> x > 100);
```

---

## 10. Collectors

Common collectors:

- `toList()`
- `toSet()`
- `toMap()`
- `joining()`
- `groupingBy()`
- `partitioningBy()`
- `counting()`
- `summarizingInt()`
- `mapping()`

### groupingBy

Think **SQL GROUP BY**.

```java
Map<String, List<Employee>> result = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
```

### groupingBy + counting

```java
Map<String, Long> result = employees.stream()
        .collect(Collectors.groupingBy(
                Employee::getDepartment,
                Collectors.counting()));
```

### partitioningBy

Splits into two groups based on a predicate.

```java
Map<Boolean, List<Employee>> result = employees.stream()
        .collect(Collectors.partitioningBy(
                e -> e.getSalary() > 50000));
```

### joining

```java
String result = names.stream()
        .collect(Collectors.joining(", "));
```

### toMap duplicate keys

Without a merge function, duplicate keys cause an exception.

```java
Collectors.toMap(
        Employee::getId,
        Employee::getName,
        (oldValue, newValue) -> oldValue
);
```

---

## 11. Optional

**Interview answer:** Optional represents a value that may or may not be present and helps make absence explicit.

```java
Optional<String> name = Optional.ofNullable(getName());
```

Important methods:

- `of()`
- `ofNullable()`
- `empty()`
- `isPresent()`
- `ifPresent()`
- `orElse()`
- `orElseGet()`
- `orElseThrow()`
- `map()`
- `filter()`

### of vs ofNullable

```java
Optional.of(null);        // NullPointerException
Optional.ofNullable(null); // Optional.empty()
```

### orElse vs orElseGet

`orElse(value)` evaluates the fallback argument eagerly.

`orElseGet(supplier)` invokes the supplier only when the Optional is empty.

Memory trick: **Optional = maybe a value**

---

## 12. Method Reference

A shorthand for certain lambda expressions using `::`.

```java
list.forEach(x -> System.out.println(x));
```

becomes:

```java
list.forEach(System.out::println);
```

Common forms:

```text
Class::staticMethod
object::instanceMethod
Class::instanceMethod
Class::new
```

Constructor reference:

```java
Supplier<Employee> s = Employee::new;
```

Memory trick: **:: = method/constructor reference**

---

## 13. Default Methods in Interfaces

Java 8 allows implementation in interfaces.

```java
interface Vehicle {
    default void start() {
        System.out.println("Starting");
    }
}
```

**Why?** To add new behavior to existing interfaces without forcing every existing implementation class to implement the new method.

---

## 14. Static Methods in Interfaces

```java
interface Test {
    static void display() {
        System.out.println("Hello");
    }
}
```

Call using the interface:

```java
Test.display();
```

---

## 15. Default Method Conflict

If two interfaces provide the same default method, the implementing class must resolve the conflict.

```java
class C implements A, B {
    @Override
    public void show() {
        A.super.show();
    }
}
```

---

## 16. Date and Time API

Java 8 introduced a better date/time API.

Main classes:

- `LocalDate`
- `LocalTime`
- `LocalDateTime`
- `ZonedDateTime`
- `Instant`
- `Period`
- `Duration`
- `DateTimeFormatter`

### LocalDate

```java
LocalDate date = LocalDate.now();
```

### LocalTime

```java
LocalTime time = LocalTime.now();
```

### LocalDateTime

```java
LocalDateTime dt = LocalDateTime.now();
```

No timezone information.

### ZonedDateTime

```java
ZonedDateTime zdt = ZonedDateTime.now(
        ZoneId.of("Asia/Kolkata"));
```

### Instant

Represents a point on the UTC timeline.

```java
Instant now = Instant.now();
```

### Period vs Duration

```text
Period   -> years / months / days
Duration -> time-based amount such as hours / minutes / seconds
```

### Formatter

```java
DateTimeFormatter formatter =
        DateTimeFormatter.ofPattern("dd-MM-yyyy");
```

---

## 17. Primitive Streams

Java 8 provides:

- `IntStream`
- `LongStream`
- `DoubleStream`

Example:

```java
int sum = IntStream.of(1, 2, 3, 4).sum();
```

Useful for primitive numeric processing and avoiding unnecessary boxing in suitable cases.

---

## 18. mapToInt()

Very common in coding interviews.

```java
int total = employees.stream()
        .mapToInt(Employee::getSalary)
        .sum();
```

---

## 19. Comparator in Java 8

```java
employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary))
        .collect(Collectors.toList());
```

Descending:

```java
employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary).reversed())
        .collect(Collectors.toList());
```

Multiple fields:

```java
employees.stream()
        .sorted(Comparator.comparing(Employee::getSalary)
                .thenComparing(Employee::getName))
        .collect(Collectors.toList());
```

Memory trick: **comparing -> thenComparing -> reversed**

---

## 20. Parallel Stream

```java
list.parallelStream();
```

Parallel streams can process elements concurrently and commonly use the common ForkJoinPool.

Important interview point:

> Parallel stream is not automatically faster. It can add overhead and is suitable only when the workload and stream characteristics benefit from parallelism.

---

## 21. Stream vs Collection

**Collection = stores/manages data**

**Stream = processes data**

A stream is consumable and generally cannot be reused after a terminal operation.

---

## 22. Stream Reuse

This is invalid:

```java
Stream<String> s = list.stream();
s.count();
s.forEach(System.out::println); // IllegalStateException
```

Create a new stream for another pipeline.

---

## 23. Lazy Evaluation

Intermediate operations are lazy.

```java
list.stream()
    .filter(x -> x > 10);
```

No terminal operation means the pipeline is not executed.

Add a terminal operation:

```java
list.stream()
    .filter(x -> x > 10)
    .count();
```

---

# ⭐ Common Java 8 Coding Questions

## Highest salary

```java
Optional<Employee> highestPaidEmployee = employees.stream()
        .max(Comparator.comparing(Employee::getSalary));
```

## Highest salary value

```java
Optional<Integer> highestSalary = employees.stream()
        .map(Employee::getSalary)
        .max(Integer::compare);
```

## Second highest salary

```java
Optional<Integer> secondHighest = employees.stream()
        .map(Employee::getSalary)
        .distinct()
        .sorted(Comparator.reverseOrder())
        .skip(1)
        .findFirst();
```

## Even numbers

```java
List<Integer> even = numbers.stream()
        .filter(n -> n % 2 == 0)
        .collect(Collectors.toList());
```

## Remove duplicates

```java
List<Integer> unique = numbers.stream()
        .distinct()
        .collect(Collectors.toList());
```

## Sort descending

```java
List<Integer> result = numbers.stream()
        .sorted(Comparator.reverseOrder())
        .collect(Collectors.toList());
```

## Uppercase

```java
List<String> result = names.stream()
        .map(String::toUpperCase)
        .collect(Collectors.toList());
```

## Group by department

```java
Map<String, List<Employee>> byDept = employees.stream()
        .collect(Collectors.groupingBy(Employee::getDepartment));
```

---

# 🔥 Rapid-Fire Interview Answers

### What are the major Java 8 features?
Lambda expressions, functional interfaces, Stream API, Optional, method references, default/static interface methods, the new Date/Time API, and related collector/functional programming improvements.

### What is a functional interface?
An interface with exactly one abstract method.

### What is Lambda?
A concise implementation of a functional interface.

### What is Stream?
A pipeline for processing data, not a data structure for storing data.

### Why are streams lazy?
Intermediate operations are deferred until a terminal operation is invoked, enabling pipeline optimization and short-circuiting.

### filter vs map?
Filter selects; map transforms.

### map vs flatMap?
Map transforms; flatMap transforms and flattens nested structures.

### Collection vs Stream?
Collection stores data; Stream processes data.

### Can a Stream be reused?
No, once a terminal operation has consumed it, a new stream must be created.

### What is Optional?
A container that represents a value that may or may not be present.

### orElse vs orElseGet?
`orElse` evaluates its fallback argument eagerly; `orElseGet` obtains it lazily from a supplier when empty.

### What is method reference?
A shorthand for certain lambdas using `::`.

### Why default methods?
To evolve interfaces without requiring every existing implementation to immediately implement newly added methods.

### Is parallel stream always faster?
No. It depends on workload, data size, splitting cost, ordering, and overhead.

---

# 🧠 FINAL 2-MINUTE MEMORY SHEET

```text
JAVA 8
│
├── Lambda          -> short implementation
├── Functional      -> one abstract method
├── Predicate       -> test
├── Function        -> transform
├── Consumer        -> consume
├── Supplier        -> supply
│
├── Stream
│   ├── filter      -> select
│   ├── map         -> transform
│   ├── flatMap     -> flatten
│   ├── distinct    -> unique
│   ├── sorted      -> sort
│   └── collect     -> result
│
├── Optional        -> maybe value
├── ::              -> method reference
├── default         -> interface implementation
├── static          -> interface utility method
├── Date/Time       -> modern date API
└── Collectors      -> group/join/map/partition
```
