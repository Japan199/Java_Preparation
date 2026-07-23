# Concurrency, JVM, Memory, and Performance

## Concurrency

### 1. Process versus thread

**Interview-ready answer:**

A process has its own memory space and resources. Threads are execution units inside a process and share heap memory, which makes communication cheaper but introduces synchronization risks. Each Java thread has its own stack, while objects are generally allocated on the shared heap.

### 2. How can you create concurrent tasks in Java?

**Interview-ready answer:**

I define work using `Runnable` when there is no result or `Callable` when a result or checked exception is needed, then submit it to an `ExecutorService`. I avoid manually creating a new `Thread` per task because executors control reuse, queueing, shutdown, and resource limits.

### 3. `Runnable` versus `Callable`

**Interview-ready answer:**

`Runnable.run()` returns no result and cannot declare checked exceptions. `Callable.call()` returns a value and may throw checked exceptions. Submitting either to an executor returns a `Future`, although a runnable’s future normally contains a null result.

### 4. What are thread states?

**Interview-ready answer:**

Java thread states are `NEW`, `RUNNABLE`, `BLOCKED`, `WAITING`, `TIMED_WAITING`, and `TERMINATED`. `RUNNABLE` includes ready and running at the Java level. `BLOCKED` normally means waiting to enter a synchronized monitor.

### 5. What does `synchronized` provide?

**Interview-ready answer:**

`synchronized` provides mutual exclusion and memory visibility around the same monitor. A synchronized instance method locks the object; a static synchronized method locks the `Class` object. Exiting the monitor happens-before a later successful entry on the same monitor.

### 6. `synchronized` versus `Lock`

**Interview-ready answer:**

`synchronized` is simple and automatically releases the monitor. `Lock`, especially `ReentrantLock`, offers timed or interruptible acquisition, fairness options, multiple conditions, and explicit lock control. With `Lock`, I always unlock in `finally`.

### 7. What does `volatile` guarantee?

**Interview-ready answer:**

`volatile` guarantees visibility of the latest write and prevents certain reorderings. It does not make compound actions such as `count++` atomic. It works well for state flags or independently written values; counters need atomic classes or locking.

### 8. Atomicity, visibility, and ordering

**Interview-ready answer:**

Atomicity means an operation is indivisible, visibility means one thread sees another thread’s updates, and ordering concerns allowed instruction reordering. Correct concurrent code must address all relevant guarantees through synchronization, volatile variables, atomics, immutable data, or concurrent utilities.

### 9. What is a race condition?

**Interview-ready answer:**

A race condition occurs when the result depends on unpredictable interleaving of threads accessing shared state. I prevent it by minimizing shared mutable state and using atomic operations or correctly scoped synchronization.

### 10. Deadlock, livelock, and starvation

**Interview-ready answer:**

Deadlock means threads wait forever for each other’s locks. Livelock means threads keep reacting but make no progress. Starvation means a thread rarely gets CPU or a required resource. I prevent deadlocks with consistent lock ordering, small critical sections, timeouts, and avoiding external calls while holding locks.

### 11. How would you diagnose a deadlock?

**Interview-ready answer:**

I capture thread dumps using tools such as `jstack`, `jcmd`, or the JVM diagnostics available in the platform, then inspect threads waiting on each other’s monitors. I correlate the locks with code paths and metrics, reproduce when possible, and fix lock ordering or redesign shared-state ownership.

### 12. `wait()`, `notify()`, and `notifyAll()`

**Interview-ready answer:**

They coordinate threads through an object monitor and must be called while holding that monitor. `wait()` releases the monitor and waits; `sleep()` does not release a lock. I check conditions in a `while` loop because of spurious wakeups. In application code, higher-level utilities are usually safer.

### 13. `sleep()` versus `wait()`

**Interview-ready answer:**

`Thread.sleep()` pauses the current thread for time and does not release held monitors. `wait()` is called on an object, releases that object’s monitor, and waits for notification, interruption, or timeout.

### 14. What is an executor?

**Interview-ready answer:**

An executor separates task submission from thread management. A thread pool bounds or manages worker threads and queues tasks. Pool sizing, queue type, rejection policy, and shutdown must match whether work is CPU-bound or I/O-bound.

### 15. How do you size a thread pool?

**Interview-ready answer:**

For CPU-bound work, a starting point is around the number of available cores. I/O-bound work can use more threads based on wait-to-compute ratio, but I validate with load tests and downstream capacity. The correct size protects latency and dependencies rather than maximizing thread count.

### 16. Why can an unbounded task queue be dangerous?

**Interview-ready answer:**

If producers outpace workers, an unbounded queue grows, increases latency, retains objects, and may cause out-of-memory failure. A bounded queue plus a deliberate rejection or backpressure strategy makes overload visible and protects the process.

### 17. What is `CompletableFuture`?

**Interview-ready answer:**

`CompletableFuture` models asynchronous stages that can be composed without blocking at every step. `thenApply` transforms a result, `thenCompose` flattens another asynchronous call, `thenCombine` joins independent results, and `exceptionally` or `handle` manages failures. I provide an appropriate executor instead of blindly using the common pool.

### 18. `thenApply()` versus `thenCompose()`

**Interview-ready answer:**

`thenApply` is like `map`: it converts `T` to `R`. If the function returns another future, it produces a nested future. `thenCompose` is like `flatMap`: it converts `T` to `CompletableFuture<R>` and flattens the result.

### 19. `Future` limitations

**Interview-ready answer:**

`Future` supports result retrieval, cancellation, and status checks, but composition and exception handling are awkward and `get()` blocks. `CompletableFuture` adds declarative composition, callbacks, and combined stages.

### 20. Explain `CountDownLatch`, `CyclicBarrier`, and `Semaphore`.

**Interview-ready answer:**

`CountDownLatch` lets threads wait until a one-time count reaches zero. `CyclicBarrier` makes a fixed group wait for each other at a reusable barrier. `Semaphore` limits concurrent access using permits, for example to protect a constrained downstream service.

### 21. What is `ThreadLocal`?

**Interview-ready answer:**

`ThreadLocal` stores a value separately per thread, often for request context in traditional thread-per-request code. In pooled threads I remove values in `finally` to prevent data leaks across requests. With virtual threads, I still avoid using thread locals as an uncontrolled global context mechanism.

### 22. What is `ConcurrentHashMap.computeIfAbsent()` useful for?

**Interview-ready answer:**

It atomically computes and installs a value when a key is absent, avoiding a race in manual get-then-put logic. The mapping function should be short and should not recursively update conflicting keys.

### 23. What is the Java Memory Model happens-before relationship?

**Interview-ready answer:**

Happens-before defines when one action’s effects must be visible to another. Examples include monitor unlock before a later lock on the same monitor, volatile write before a later read of that variable, actions before `Thread.start()`, and thread actions before another thread successfully returns from `join()`.

## JVM and Memory

### 24. Explain JVM runtime memory areas.

**Interview-ready answer:**

The heap stores most objects and is shared. Each thread has a stack containing frames, local variables, and calls. Metaspace stores class metadata in native memory. There are also the program counter and native method stacks. The exact implementation can optimize allocations, but this is the useful conceptual model.

### 25. Stack versus heap

**Interview-ready answer:**

Stacks are per-thread and automatically manage method frames; deep recursion can cause `StackOverflowError`. The heap is shared and garbage-collected; excessive retained objects can cause `OutOfMemoryError`. A local variable may be on a stack while the object it references is on the heap.

### 26. What is Metaspace?

**Interview-ready answer:**

Metaspace stores class metadata in native memory and replaced PermGen. It can grow until constrained by configuration or native memory. Class-loader leaks can exhaust it, especially in environments that repeatedly deploy or dynamically generate classes.

### 27. How does garbage collection work?

**Interview-ready answer:**

GC identifies objects that are not reachable from GC roots and reclaims their memory. Generational collectors use the observation that most objects die young, separating young and old regions or generations. Collection behavior, pauses, and compaction depend on the selected collector.

### 28. What are GC roots?

**Interview-ready answer:**

GC roots are starting points for reachability analysis, such as live thread stacks, static references, JNI references, and certain JVM internal references. An object reachable from a root is considered live even if the application no longer logically needs it.

### 29. Can Java have memory leaks?

**Interview-ready answer:**

Yes. GC frees unreachable objects, but objects accidentally kept reachable still leak. Common causes are unbounded caches, static collections, listeners not removed, thread locals in pools, unclosed resources, and class-loader retention.

### 30. How do you investigate high memory usage?

**Interview-ready answer:**

I confirm the symptom using heap and GC metrics, capture a heap dump safely, and inspect the dominator tree, retained sizes, and paths to GC roots with tools such as Eclipse MAT or JDK tooling. I compare dumps over time, find the retaining owner, fix lifecycle or bounds, and verify under load.

### 31. `OutOfMemoryError` versus `StackOverflowError`

**Interview-ready answer:**

`OutOfMemoryError` means the JVM cannot allocate in a required memory area, such as heap, Metaspace, or native threads. `StackOverflowError` usually means a thread exhausted its stack through deep or infinite recursion. The correct fix depends on the specific error message and diagnostics.

### 32. What is JIT compilation?

**Interview-ready answer:**

The JVM initially interprets or compiles code and uses runtime profiling to JIT-compile hot methods into optimized native code. This enables optimizations based on actual execution, such as inlining and escape analysis. It also explains JVM warm-up effects in benchmarks.

### 33. Why are microbenchmarks difficult?

**Interview-ready answer:**

JIT warm-up, dead-code elimination, constant folding, GC, and CPU effects can make naive timing misleading. I use JMH for Java microbenchmarks and still validate application-level behavior with realistic load tests.

### 34. Strong, soft, weak, and phantom references

**Interview-ready answer:**

Strong references keep objects alive normally. Weak references are cleared when no strong reachability remains. Soft references may survive until memory pressure but are not a reliable cache policy. Phantom references work with a reference queue for post-mortem cleanup coordination; explicit resource management remains preferable.

## Production scenario

**Question: CPU is high and API latency increased. What do you do?**

**Interview-ready answer:**

I first check whether the issue is CPU saturation, GC, traffic, or a downstream delay using metrics. For CPU saturation I capture multiple thread dumps or a short profiling recording and identify consistently hot threads and methods. I correlate them with recent deployments and request traces, mitigate through scaling or traffic control if needed, fix the hot path, and validate using load tests and production monitoring.

