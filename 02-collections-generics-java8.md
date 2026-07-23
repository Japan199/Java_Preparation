# Collections, Generics, Java 8+, and Streams

## Collections Framework

### 1. Explain the Java Collections hierarchy.

**Interview-ready answer:**

`Collection` is the root interface for `List`, `Set`, and `Queue`. `Map` is separate because it stores key-value pairs. Lists preserve position and allow duplicates, sets enforce uniqueness, queues support processing order, and maps provide lookup by key.

### 2. `ArrayList` versus `LinkedList`

**Interview-ready answer:**

`ArrayList` uses a resizable array, gives fast indexed access, and is usually the best general-purpose list. `LinkedList` uses nodes and has poor random access plus higher memory overhead. Although insertion at a known node is constant time, finding that position is linear, so `ArrayList` often performs better in practice.

### 3. How does `ArrayList` grow?

**Interview-ready answer:**

When capacity is insufficient, `ArrayList` allocates a larger internal array and copies elements. Appending is amortized O(1), while a resize is O(n). If I know the approximate size, I provide initial capacity to reduce reallocations.

### 4. `HashSet`, `LinkedHashSet`, and `TreeSet`

**Interview-ready answer:**

`HashSet` gives average O(1) operations with no ordering guarantee. `LinkedHashSet` preserves insertion order with extra memory. `TreeSet` keeps elements sorted using a tree and gives O(log n) operations. The choice depends on whether I need speed, insertion order, or sorted order.

### 5. How does `HashMap` work?

**Interview-ready answer:**

`HashMap` calculates a hash from the key, maps it to a bucket, and uses `equals()` to find the exact key within that bucket. Collisions share a bucket. In modern Java, a heavily populated bucket can become a balanced tree when conditions are met, improving worst-case lookup. Resizing occurs when size exceeds capacity times load factor.

### 6. Why should a `HashMap` key be immutable?

**Interview-ready answer:**

If fields used by `hashCode()` or `equals()` change after insertion, the key may remain in the old bucket and become effectively unreachable. Immutable keys keep identity stable and make map behavior predictable.

### 7. How does `HashMap` handle null?

**Interview-ready answer:**

`HashMap` allows one null key and multiple null values. `Hashtable` and `ConcurrentHashMap` do not allow null keys or values. `ConcurrentHashMap` rejects null partly because null would make an absent mapping ambiguous during concurrent operations.

### 8. `HashMap` versus `Hashtable` versus `ConcurrentHashMap`

**Interview-ready answer:**

`HashMap` is not thread-safe. `Hashtable` synchronizes most operations on the whole structure and is a legacy class. `ConcurrentHashMap` supports thread-safe access with much better concurrency using fine-grained coordination and lock-free techniques for many reads. For compound actions, I use atomic methods such as `computeIfAbsent()` rather than check-then-act code.

### 9. What is the load factor?

**Interview-ready answer:**

The load factor controls when a hash table resizes. `HashMap` defaults to 0.75, a balance between memory usage and collisions. Initial capacity should consider expected entries and load factor when resize cost matters.

### 10. `Comparable` versus `Comparator`

**Interview-ready answer:**

`Comparable` defines a type’s natural ordering through `compareTo()`. `Comparator` defines external or multiple orderings and can be composed. I use `Comparator.comparing()` for flexible sorting without putting every business sort rule into the domain class.

```java
employees.sort(
    Comparator.comparing(Employee::getDepartment)
              .thenComparing(Employee::getSalary, Comparator.reverseOrder())
);
```

### 11. Fail-fast versus fail-safe iteration

**Interview-ready answer:**

Fail-fast iterators detect structural modification and usually throw `ConcurrentModificationException`; this is best-effort bug detection, not a thread-safety guarantee. Concurrent collections provide weakly consistent or snapshot-style iteration instead of failing. For example, `CopyOnWriteArrayList` iterates over a snapshot.

### 12. Why does removing inside a for-each loop fail?

**Interview-ready answer:**

The enhanced for loop uses an iterator internally. Modifying the collection directly can invalidate the iterator. I use `Iterator.remove()`, `removeIf()`, or collect items to remove.

### 13. When would you use `CopyOnWriteArrayList`?

**Interview-ready answer:**

It is suitable for small collections with many reads and very few writes, such as listener lists. Reads and iteration are lock-free over a stable snapshot, but each write copies the underlying array, so it is expensive for write-heavy workloads.

### 14. Queue, Deque, and PriorityQueue

**Interview-ready answer:**

`Queue` normally models FIFO processing. `Deque` supports insertion and removal at both ends and is also preferred over legacy `Stack`. `PriorityQueue` returns elements according to priority, not insertion order, with O(log n) insertion/removal and O(1) access to the head.

### 15. Big-O summary

| Structure | Get/search | Add | Remove | Ordering |
|---|---:|---:|---:|---|
| `ArrayList` | O(1) by index, O(n) search | amortized O(1) at end | O(n) | insertion |
| `LinkedList` | O(n) | O(1) at known end/node | O(1) at known node | insertion |
| `HashSet` | average O(1) | average O(1) | average O(1) | none |
| `TreeSet` | O(log n) | O(log n) | O(log n) | sorted |
| `HashMap` | average O(1) | average O(1) | average O(1) | none |
| `TreeMap` | O(log n) | O(log n) | O(log n) | key-sorted |

## Generics

### 16. Why use generics?

**Interview-ready answer:**

Generics provide compile-time type safety, remove many casts, and allow reusable algorithms and containers. `List<String>` communicates intent and prevents adding unrelated types.

### 17. What is type erasure?

**Interview-ready answer:**

Java implements most generics through type erasure. Generic type information is checked at compile time and mostly removed or replaced by bounds in bytecode, with casts inserted where needed. Therefore I cannot normally create `new T()`, use `T.class`, or test `instanceof List<String>`.

### 18. Explain `? extends T` and `? super T`.

**Interview-ready answer:**

I use PECS: **Producer Extends, Consumer Super**. If a structure produces `T` values, `? extends T` lets me read them safely but not add a specific subtype. If it consumes `T`, `? super T` lets me add `T` values while reads are only safely treated as `Object`.

### 19. Why is `List<Integer>` not a subtype of `List<Number>`?

**Interview-ready answer:**

Generics are invariant. If it were allowed, code could add a `Double` through the `List<Number>` reference and corrupt a `List<Integer>`. Wildcards express safe variance where required.

## Lambdas and Functional Programming

### 20. What is a functional interface?

**Interview-ready answer:**

A functional interface has exactly one abstract method and can be implemented with a lambda or method reference. It may still have default and static methods. Common examples are `Predicate`, `Function`, `Consumer`, and `Supplier`.

### 21. Explain common functional interfaces.

| Interface | Input/output | Typical use |
|---|---|---|
| `Predicate<T>` | T → boolean | filtering |
| `Function<T,R>` | T → R | mapping |
| `Consumer<T>` | T → void | side effect |
| `Supplier<T>` | () → T | lazy creation |
| `UnaryOperator<T>` | T → T | same-type transformation |
| `BinaryOperator<T>` | (T,T) → T | reduction |

### 22. Lambda versus anonymous class

**Interview-ready answer:**

A lambda is a concise implementation of a functional interface and does not introduce a separate `this`; `this` refers to the enclosing object. An anonymous class creates its own scope and can implement a type with more structure. I use lambdas for small behavior and named classes when logic needs identity, state, or substantial testing.

### 23. What variables can a lambda capture?

**Interview-ready answer:**

A lambda can capture local variables that are final or effectively final. It can also access instance and static fields. The effectively-final rule avoids confusing mutation of stack-local state across delayed execution.

### 24. What are method references?

**Interview-ready answer:**

Method references are compact lambdas that delegate to an existing method: `Type::staticMethod`, `object::instanceMethod`, `Type::instanceMethod`, or `Type::new`. I use them when they improve readability, not just to make code shorter.

## Streams

### 25. Collection versus Stream

**Interview-ready answer:**

A collection stores data; a stream describes a pipeline of computation over data. Streams are lazy, normally single-use, and may process sequentially or in parallel. Intermediate operations build the pipeline and a terminal operation triggers it.

### 26. Intermediate versus terminal operations

**Interview-ready answer:**

Intermediate operations such as `filter`, `map`, `sorted`, and `distinct` return another stream and are lazy. Terminal operations such as `collect`, `reduce`, `count`, and `forEach` consume the stream and produce a result or side effect.

### 27. `map()` versus `flatMap()`

**Interview-ready answer:**

`map()` transforms each element into one result. `flatMap()` transforms each element into a stream or container and flattens the nested results. For example, it converts `List<List<String>>` into a stream of strings.

### 28. `filter()` versus `map()`

**Interview-ready answer:**

`filter()` keeps or removes elements based on a predicate without changing their type. `map()` transforms each element into another value and may change the type.

### 29. `reduce()` versus `collect()`

**Interview-ready answer:**

`reduce()` combines elements into one immutable result such as a sum. `collect()` performs mutable reduction into containers such as lists, maps, or grouped results. I use `collect()` for aggregation structures and `reduce()` for associative value combination.

### 30. Find duplicate elements using streams.

```java
Set<Integer> seen = new HashSet<>();
Set<Integer> duplicates = numbers.stream()
    .filter(n -> !seen.add(n))
    .collect(Collectors.toSet());
```

**Interview note:** This uses mutable state and is not suitable for a parallel stream. A frequency map is safer for reusable code.

### 31. Group employees by department.

```java
Map<String, List<Employee>> byDepartment = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

### 32. Find the highest-paid employee per department.

```java
Map<String, Optional<Employee>> result = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))
    ));
```

### 33. Why can stream operations be lazy?

**Interview-ready answer:**

Laziness lets the runtime combine operations, avoid intermediate collections, and short-circuit. For example, `filter(...).findFirst()` can stop after finding the first match instead of evaluating the entire source.

### 34. When should you avoid parallel streams?

**Interview-ready answer:**

I avoid parallel streams for small datasets, blocking I/O, ordered or stateful operations, shared mutable state, or request-processing code where the common ForkJoinPool is a shared resource. I use them only after measurement, when work is CPU-bound, independent, sufficiently large, and safely splittable.

### 35. `findFirst()` versus `findAny()`

**Interview-ready answer:**

`findFirst()` respects encounter order. `findAny()` may return any matching element and can allow more optimization in parallel pipelines. In sequential ordered streams they may appear identical, but their contracts differ.

### 36. What is `Optional` for?

**Interview-ready answer:**

`Optional` explicitly models a possibly absent return value and encourages deliberate handling. I mainly use it as a return type, not for entity fields, DTO fields, or method parameters. I prefer `orElseGet()` when the fallback is expensive because `orElse()` evaluates its argument eagerly.

### 37. `orElse()` versus `orElseGet()`

**Interview-ready answer:**

`orElse(value)` computes `value` before the call even when the optional is present. `orElseGet(supplier)` evaluates lazily only when empty. The result is the same, but cost and side effects may differ.

## Modern Java topics

### 38. What are default methods?

**Interview-ready answer:**

Default methods let interfaces add behavior without immediately breaking existing implementations. If two interfaces provide conflicting defaults, the implementing class must resolve the conflict explicitly. Class methods take precedence over interface defaults.

### 39. What are sealed classes?

**Interview-ready answer:**

Sealed classes and interfaces restrict which types may extend or implement them using `permits`. They are useful for closed domain hierarchies and exhaustive pattern matching. Permitted subclasses must be final, sealed, or non-sealed.

### 40. What is pattern matching?

**Interview-ready answer:**

Pattern matching combines a type test with safe variable binding, reducing casts. Modern Java also supports pattern matching for switch, which is useful with sealed hierarchies and can enforce exhaustive handling.

```java
if (value instanceof String text) {
    return text.length();
}
```

### 41. What are virtual threads?

**Interview-ready answer:**

Virtual threads are lightweight JVM-managed threads designed for high-concurrency, mostly blocking I/O workloads. They allow a simple thread-per-request style without requiring one expensive platform thread per task. They improve scalability, not the speed of CPU-bound work, and thread-local usage or pinned blocking operations still require attention.

## Common traps

- Giving `LinkedList` as automatically faster for all insertions
- Assuming `HashMap` is ordered or thread-safe
- Mutating a key after it enters a hash map
- Reusing a stream after a terminal operation
- Adding side effects to parallel stream operations
- Calling `Optional.get()` without checking
- Using parallel streams without measuring

