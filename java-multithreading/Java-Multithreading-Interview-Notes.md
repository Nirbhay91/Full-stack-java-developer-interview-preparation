# Java Multithreading — Interview Notes

## 1. What is Multithreading?

> Multithreading is the execution of multiple threads concurrently within a process. It helps improve responsiveness and resource utilization, especially when tasks can run independently.

**Memory trick:** Process = application | Thread = execution unit

---

## 2. Process vs Thread

| Process | Thread |
|---|---|
| Independent program execution | Execution unit inside process |
| Has separate memory | Shares process memory |
| Heavyweight | Lightweight |
| Communication is comparatively expensive | Communication is easier |

**Memory trick:** Process = House, Threads = People inside the house.

---

## 3. Thread vs Runnable

### Extending Thread

```java
class MyThread extends Thread {
    public void run() {
        System.out.println("Running");
    }
}
```

### Implementing Runnable

```java
class MyTask implements Runnable {
    public void run() {
        System.out.println("Running");
    }
}
```

Prefer **Runnable** because Java supports single class inheritance, so extending `Thread` unnecessarily consumes the inheritance option.

**Interview line:** "I generally prefer Runnable because it separates the task from the thread and allows the class to extend another class."

---

## 4. start() vs run()

### `start()`

Creates a new thread and JVM eventually invokes `run()` on that thread.

```java
thread.start();
```

### `run()`

Normal method call; it does **not** create a new thread.

```java
thread.run();
```

**Memory trick:** start = new thread | run = normal method call

---

## 5. Thread Life Cycle

Java's official `Thread.State` values are:

```text
NEW
RUNNABLE
BLOCKED
WAITING
TIMED_WAITING
TERMINATED
```

Important: Java does **not** have separate `RUNNING` and `READY` states in `Thread.State`.

---

## 6. sleep()

```java
Thread.sleep(1000);
```

It pauses the current thread for a specified duration.

Important: **`sleep()` does NOT release a lock/monitor that the thread already holds.**

**Memory trick:** Sleep = Pause, keep lock

---

## 7. yield()

```java
Thread.yield();
```

It is a scheduler hint suggesting that the current thread is willing to give another runnable thread a chance to execute.

Important: **`yield()` does not release a held monitor lock and does not guarantee that another thread will run.** The thread remains `RUNNABLE`.

**Memory trick:** yield = Request, not guarantee

---

## 8. wait()

`wait()` is used for inter-thread communication.

```java
synchronized (lock) {
    lock.wait();
}
```

When a thread calls `wait()`:

1. It enters waiting state.
2. **Releases the object's monitor.**
3. Waits until another thread calls `notify()`/`notifyAll()` or it is otherwise awakened.

**Memory trick:** wait = Release lock + wait

---

## 9. notify()

```java
synchronized (lock) {
    lock.notify();
}
```

Wakes **one** thread waiting on that object's monitor.

---

## 10. notifyAll()

```java
synchronized (lock) {
    lock.notifyAll();
}
```

Wakes **all** threads waiting on that object's monitor.

`wait()`, `notify()`, and `notifyAll()` must be called while holding the object's monitor, normally inside `synchronized`.

**Memory trick:** wait → release | notify → wake one | notifyAll → wake all

---

## 11. synchronized

Used to control access to shared resources so that only one thread at a time executes the synchronized critical section for the same monitor.

```java
public synchronized void increment() {
    count++;
}
```

It provides:
- Mutual exclusion
- Visibility guarantees associated with monitor locking

**Memory trick:** synchronized = One-at-a-time access

---

## 12. Synchronized Method vs Block

### Method

```java
public synchronized void method() {
}
```

Locks the object's monitor for an instance method.

### Block

```java
synchronized (lock) {
    // critical section
}
```

Allows control over which object is used as the lock and can minimize the locked region.

**Memory trick:** Block = More control

---

## 13. Static synchronized

```java
public static synchronized void test() {
}
```

Instance synchronized method locks the **object instance**.

Static synchronized method locks the **Class object's monitor**.

**Memory trick:** instance → object lock | static → class lock

---

## 14. Race Condition

A race condition occurs when multiple threads access shared mutable data concurrently and the result depends on the timing/interleaving of their operations.

Example:

```java
count++;
```

Multiple threads can read and update the same value incorrectly.

**Memory trick:** Race = Threads competing for shared data

---

## 15. Thread Safety

A class/code is thread-safe when it behaves correctly under concurrent access.

Common techniques:
- Synchronization
- Atomic classes
- Immutable objects
- Concurrent collections
- Proper locking

---

## 16. Atomicity

An operation is atomic when it happens as one indivisible operation from the relevant concurrency perspective.

Example:

```java
count++;
```

is **not atomic**.

It involves:

```text
read → increment → write
```

**Memory trick:** Atomic = Can't be partially observed/interleaved as separate steps

---

## 17. volatile

> `volatile` provides **visibility guarantees** for a variable across threads.

```java
private volatile boolean running = true;
```

If one thread changes it, other threads can observe the update without relying on a stale cached value.

Important: `volatile` does **NOT** make compound operations atomic.

```java
count++;
```

is still unsafe.

**Memory trick:** volatile = Visibility, NOT Atomicity

---

## 18. volatile vs synchronized

| volatile | synchronized |
|---|---|
| Mainly visibility/order guarantees | Mutual exclusion + visibility |
| No locking | Uses monitor locking |
| Doesn't make `count++` atomic | Can protect compound operations |
| Good for simple state flags | Good for critical sections |

---

## 19. Deadlock

Deadlock occurs when threads wait indefinitely for locks held by each other.

```text
Thread 1 → Lock A → waits for B
Thread 2 → Lock B → waits for A
```

Both wait forever.

**Memory trick:** A wants B, B wants A = Deadlock

### Prevention
- Consistent lock ordering
- Avoid unnecessary nested locks
- Use timed locking where appropriate
- Keep critical sections small

---

## 20. Starvation

A thread is starved when it doesn't get enough opportunity to execute because other threads continually consume the available resources/CPU/locks.

**Memory trick:** Starvation = Thread keeps waiting for chance

---

## 21. Livelock

Threads are active but keep responding to each other without making useful progress.

**Memory trick:** Deadlock = doing nothing | Livelock = doing something, but no progress

---

## 22. Thread Pool

A Thread Pool is a collection of reusable worker threads.

Instead of:

```text
Create thread → execute → destroy → create another
```

we do:

```text
Task → Thread Pool → Available Worker Thread → Execute → Return to Pool
```

Benefits:
- Reuse threads
- Reduce creation overhead
- Control concurrency
- Improve resource management

**Memory trick:** Thread Pool = Create once, reuse many times

---

## 23. ExecutorService

Java provides the Executor framework to manage task execution.

```java
ExecutorService executor =
        Executors.newFixedThreadPool(5);

executor.submit(() -> {
    System.out.println("Task");
});

executor.shutdown();
```

Important methods:

```text
execute()
submit()
shutdown()
shutdownNow()
```

---

## 24. execute() vs submit()

### execute()

```java
executor.execute(task);
```

Used for executing a `Runnable`. It doesn't return a `Future`.

### submit()

```java
Future<?> future = executor.submit(task);
```

Can accept `Runnable` or `Callable` and returns a `Future`.

**Memory trick:** submit = result/future

---

## 25. Callable vs Runnable

### Runnable

- `run()`
- No return value
- Cannot directly throw checked exceptions

### Callable

```java
Callable<Integer> c = () -> {
    return 10;
};
```

- `call()`
- Returns a value
- Can throw checked exceptions

**Memory trick:** Runnable = Run | Callable = Calculate + Return

---

## 26. Future

`Future` represents the result of an asynchronous computation.

```java
Future<Integer> future =
        executor.submit(() -> 10);

Integer result = future.get();
```

`get()` can block until the result is available.

---

## 27. CompletableFuture

Used for asynchronous programming and composing asynchronous tasks.

```java
CompletableFuture
    .supplyAsync(() -> getUser())
    .thenApply(user -> user.getName())
    .thenAccept(System.out::println);
```

Useful for:
- Async execution
- Chaining
- Combining tasks
- Error handling

**Memory trick:** CompletableFuture = Async + Chain + Combine

---

## 28. Atomic Classes

Package:

```java
java.util.concurrent.atomic
```

Examples:

```text
AtomicInteger
AtomicLong
AtomicBoolean
```

Example:

```java
AtomicInteger count = new AtomicInteger();
count.incrementAndGet();
```

Useful for atomic updates without manually synchronizing simple operations.

---

## 29. Concurrent Collections

Examples:

```text
ConcurrentHashMap
CopyOnWriteArrayList
BlockingQueue
```

### ConcurrentHashMap

Designed for concurrent access to a map with better scalability than synchronizing the entire map for every operation.

---

## 30. BlockingQueue

A thread-safe queue designed for producer-consumer scenarios.

```text
Producer
   ↓
BlockingQueue
   ↓
Consumer
```

Common implementations:
- `ArrayBlockingQueue`
- `LinkedBlockingQueue`

**Memory trick:** Producer puts → Consumer takes

---

## 31. Producer-Consumer Problem

Producer generates data. Consumer processes data.

`BlockingQueue` is a common solution:

```java
queue.put(data);
queue.take();
```

If the queue is full, `put()` can wait.

If the queue is empty, `take()` can wait.

---

## 32. ReentrantLock

Alternative to intrinsic `synchronized`.

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    // critical section
} finally {
    lock.unlock();
}
```

Advantages include:
- Explicit locking/unlocking
- `tryLock()`
- Interruptible locking
- Optional fairness configuration

**Memory trick:** ReentrantLock = More lock control

---

## 33. synchronized vs ReentrantLock

| synchronized | ReentrantLock |
|---|---|
| Simpler | More flexible |
| Automatic monitor release when leaving block/method | Must explicitly unlock |
| No `tryLock()` | Has `tryLock()` |
| Built into language | Explicit API |
| Good default choice | Useful for advanced locking needs |

---

## 34. Semaphore

A `Semaphore` controls access to a resource using a number of permits.

```java
Semaphore semaphore = new Semaphore(3);
```

At most 3 permits can be acquired concurrently.

**Memory trick:** Semaphore = Limited seats

Example: only 3 threads can access a limited external resource at once.

---

## 35. CountDownLatch

Allows one or more threads to wait until a counter reaches zero.

```java
CountDownLatch latch = new CountDownLatch(3);
```

```text
Worker 1 → countDown()
Worker 2 → countDown()
Worker 3 → countDown()
                 ↓
              count = 0
                 ↓
         waiting thread continues
```

**Memory trick:** Latch = Wait until N tasks finish

---

## 36. CyclicBarrier

Allows a fixed number of threads to wait for each other at a common barrier.

```text
Thread 1 ─┐
Thread 2 ─┼→ Barrier → Continue
Thread 3 ─┘
```

**Memory trick:** Barrier = Everyone arrives → everyone continues

### Latch vs Barrier

`CountDownLatch` is generally one-time use.

`CyclicBarrier` can be reused for multiple phases.

---

## 37. ReadWriteLock

Useful when **many reads + fewer writes**.

Multiple readers can read concurrently, but writing requires exclusive access.

```java
ReadWriteLock lock = new ReentrantReadWriteLock();
```

**Memory trick:** Many readers, one writer

---

## 38. ThreadLocal

Provides a separate value for each thread.

```java
ThreadLocal<Integer> local =
        ThreadLocal.withInitial(() -> 0);
```

Each thread gets its own independent value.

Useful for thread-confined context such as request-related data.

**Memory trick:** ThreadLocal = One value per thread

---

## 39. Context Switching

CPU switches from one thread to another. This has overhead because the system needs to save and restore execution state.

**Important:** More threads ≠ always more performance.

Too many threads can cause excessive context switching and resource consumption.

---

## 40. Inter-thread Communication

Main mechanisms:

```text
wait()
notify()
notifyAll()
```

Modern applications also commonly use:

```text
BlockingQueue
CountDownLatch
CyclicBarrier
Semaphore
CompletableFuture
```

---

# Most Important Interview Questions

### Basic
1. What is multithreading?
2. Process vs Thread?
3. Thread vs Runnable?
4. `start()` vs `run()`?
5. Thread lifecycle/states?

### Synchronization
6. What is `synchronized`?
7. Method vs block synchronization?
8. What is race condition?
9. What is thread safety?
10. What is atomicity?

### Communication
11. `wait()` vs `sleep()`?
12. `wait()` vs `notify()`?
13. `notify()` vs `notifyAll()`?
14. Why must wait/notify be called inside synchronized context?
15. Does `sleep()` release the lock?
16. Does `wait()` release the lock?
17. Does `yield()` release the lock?

### Visibility
18. What is `volatile`?
19. Why doesn't volatile make `count++` thread-safe?
20. volatile vs synchronized?

### Problems
21. Deadlock?
22. Starvation?
23. Livelock?
24. How do you prevent deadlock?

### Concurrency utilities
25. Thread Pool?
26. ExecutorService?
27. `execute()` vs `submit()`?
28. Runnable vs Callable?
29. Future?
30. CompletableFuture?
31. AtomicInteger?
32. ConcurrentHashMap?
33. BlockingQueue?
34. ReentrantLock?
35. Semaphore?
36. CountDownLatch?
37. CyclicBarrier?
38. ThreadLocal?

---

# Ultimate Revision Map

```text
MULTITHREADING
│
├── Thread
│   ├── start()
│   ├── run()
│   ├── sleep()
│   └── yield()
│
├── Synchronization
│   ├── synchronized
│   ├── Lock
│   └── ReentrantLock
│
├── Communication
│   ├── wait()
│   ├── notify()
│   └── notifyAll()
│
├── Visibility
│   └── volatile
│
├── Atomicity
│   └── AtomicInteger
│
├── Execution
│   ├── ExecutorService
│   ├── Thread Pool
│   ├── Runnable
│   ├── Callable
│   └── Future
│
├── Async
│   └── CompletableFuture
│
├── Concurrent Collections
│   ├── ConcurrentHashMap
│   └── BlockingQueue
│
├── Coordination
│   ├── CountDownLatch
│   ├── CyclicBarrier
│   └── Semaphore
│
└── Problems
    ├── Race Condition
    ├── Deadlock
    ├── Starvation
    └── Livelock
```

# 10-Line Interview Revision

> **Thread = execution unit.**  
> **start() creates concurrent execution; run() is just a method call.**  
> **sleep() pauses but doesn't release the monitor.**  
> **wait() releases the monitor and waits.**  
> **notify() wakes one waiting thread; notifyAll() wakes all.**  
> **synchronized provides mutual exclusion and visibility guarantees.**  
> **volatile provides visibility, not atomicity.**  
> **Race condition happens due to unsynchronized shared mutable state.**  
> **Deadlock means threads wait indefinitely for each other's locks.**  
> **Thread Pool reuses threads and controls concurrency.**
