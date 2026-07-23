# Java Fundamentals and OOP

## 1. What are the main features of Java?

**Interview-ready answer:**

Java is object-oriented, strongly typed, platform-independent through JVM bytecode, garbage-collected, multithreaded, and has a rich standard library. It is also robust because it provides exception handling, type checking, and automatic memory management. “Write once, run anywhere” means compiled bytecode can run on any compatible JVM.

## 2. Explain JDK, JRE, and JVM.

**Interview-ready answer:**

- **JVM** executes Java bytecode and manages runtime concerns such as memory and garbage collection.
- **JRE** contains the JVM and libraries needed to run Java applications.
- **JDK** contains the JRE plus development tools such as `javac`, `javadoc`, and debuggers.

In modern Java distributions, a separate public JRE is less common, but the conceptual distinction is still useful.

## 3. Why is Java platform-independent?

**Interview-ready answer:**

Java source code is compiled into platform-neutral bytecode. A platform-specific JVM translates or JIT-compiles that bytecode into native instructions. The same `.class` file can therefore run on Windows, Linux, or macOS when a compatible JVM is installed.

## 4. What are the four pillars of OOP?

**Interview-ready answer:**

- **Encapsulation:** keep state private and expose controlled behavior.
- **Abstraction:** expose essential behavior and hide implementation details.
- **Inheritance:** derive behavior from an existing type when there is a genuine “is-a” relationship.
- **Polymorphism:** use a common contract while runtime objects provide different implementations.

In production code I generally favor composition over inheritance because it reduces coupling.

## 5. Abstraction versus encapsulation

**Interview-ready answer:**

Abstraction focuses on **what** an object does, usually through an interface or abstract class. Encapsulation focuses on **how** state and implementation are protected, usually using private fields and methods. For example, a `PaymentProcessor` interface provides abstraction, while its implementation encapsulates credentials and API-call logic.

## 6. Interface versus abstract class

**Interview-ready answer:**

I use an interface to define a capability or contract that unrelated classes can implement. I use an abstract class when related classes need shared state, constructors, or common implementation. Interfaces support multiple inheritance of type and can contain default and static methods, while a class can extend only one class.

## 7. Overloading versus overriding

**Interview-ready answer:**

Overloading means the same method name has different parameter lists and is resolved at compile time. Overriding means a subclass provides a new implementation with the same compatible signature and is resolved at runtime. Return type alone cannot overload a method. An overriding method cannot reduce visibility or throw broader checked exceptions.

## 8. Can static, private, and final methods be overridden?

**Interview-ready answer:**

No. Static methods are hidden, not overridden, because they belong to the class. Private methods are not visible to subclasses. Final methods explicitly prevent overriding.

## 9. What is the difference between `==` and `equals()`?

**Interview-ready answer:**

For primitives, `==` compares values. For objects, `==` compares references, while `equals()` compares logical equality if the class overrides it. For example, two different `String` objects may be equal according to `equals()` but not identical according to `==`.

## 10. Explain the `equals()` and `hashCode()` contract.

**Interview-ready answer:**

Equal objects must return the same hash code. Unequal objects may still have the same hash code because collisions are allowed. `equals()` should be reflexive, symmetric, transitive, consistent, and return false for null. Whenever I override `equals()`, I also override `hashCode()`, especially for objects used as keys in hash-based collections.

## 11. Why is `String` immutable?

**Interview-ready answer:**

String immutability provides thread safety, safe sharing through the string pool, stable hash codes for map keys, and security for values such as class names or URLs. Operations that appear to modify a string actually create a new object.

## 12. `String`, `StringBuilder`, and `StringBuffer`

**Interview-ready answer:**

`String` is immutable. `StringBuilder` is mutable and is the normal choice for repeated concatenation in a single thread. `StringBuffer` is mutable and synchronized, so it is thread-safe but usually slower. For building a string inside a method, I use `StringBuilder`.

## 13. What is the String pool?

**Interview-ready answer:**

The string pool stores canonical string literals so identical literals can share one object. `new String("java")` creates a separate heap object, while `"java"` may reuse the pooled instance. Calling `intern()` returns the canonical pooled representation, but I avoid unnecessary manual interning because it adds global-pool overhead.

## 14. Is Java pass-by-reference?

**Interview-ready answer:**

No, Java is always pass-by-value. For an object, the value being copied is the reference. A method can mutate the object through that copied reference, but reassigning the parameter does not change the caller’s reference.

```java
void change(User user) {
    user.setName("A");     // visible to caller
    user = new User("B");  // reassignment is not visible to caller
}
```

## 15. Primitive types versus wrapper classes

**Interview-ready answer:**

Primitives store simple values and cannot be null. Wrappers are objects required by generics and collections and can be null. Autoboxing converts a primitive to a wrapper and unboxing does the reverse. I take care with wrapper comparison and null unboxing because `Integer == Integer` may compare references and unboxing null throws `NullPointerException`.

## 16. Explain the Integer cache.

**Interview-ready answer:**

Java normally caches `Integer` objects from -128 to 127. Autoboxed values in that range may have the same reference, so `==` can appear to work. Outside the range it may return false. I use `equals()` or compare unboxed primitive values instead of depending on the cache.

## 17. `final`, `finally`, and `finalize`

**Interview-ready answer:**

- `final` prevents reassignment, overriding, or inheritance depending on context.
- `finally` is a block normally executed after `try`/`catch` for cleanup.
- `finalize()` was an unreliable GC hook and is deprecated for removal; resources should use try-with-resources or explicit lifecycle management.

## 18. Does `final` make an object immutable?

**Interview-ready answer:**

No. A final reference cannot point to another object, but the referenced object may still be mutable. An immutable class also needs private final fields, no mutating methods, controlled construction, and defensive copies of mutable data.

## 19. How do you create an immutable class?

**Interview-ready answer:**

I make the class final, make fields private and final, initialize them fully in the constructor, provide no setters, validate inputs, and defensively copy mutable inputs and outputs. Java records help with concise data carriers, but a record is only shallowly immutable if its components include mutable objects.

## 20. What is a record?

**Interview-ready answer:**

A record is a concise way to model transparent data carriers. The compiler provides final fields, accessors, a canonical constructor, `equals()`, `hashCode()`, and `toString()`. Records are final and useful for DTOs or value objects, but they should not usually contain mutable components when value semantics matter.

## 21. What is an enum and why use it?

**Interview-ready answer:**

An enum represents a fixed set of type-safe constants. Unlike integer or string constants, enums can have fields, constructors, and methods. I use them for states such as `PENDING`, `PAID`, and `FAILED`, often with behavior to avoid large conditional blocks.

## 22. Composition versus inheritance

**Interview-ready answer:**

Inheritance creates a strong “is-a” relationship and couples a subclass to parent behavior. Composition builds a class using collaborators and supports changing behavior through interfaces. I prefer composition for most business services because it is easier to test and evolve; I use inheritance only for a stable, meaningful hierarchy.

## 23. Association, aggregation, and composition

**Interview-ready answer:**

Association is a general relationship between objects. Aggregation is a weak whole-part relationship where the part can exist independently. Composition is a strong ownership relationship where the part’s lifecycle belongs to the whole. For example, an order and customer are associated, while order lines are normally composed within an order.

## 24. What are access modifiers?

**Interview-ready answer:**

`private` is visible only inside the class. Package-private is visible within the package. `protected` adds visibility to subclasses, including subclasses in other packages under inheritance rules. `public` is visible everywhere. I expose the smallest surface needed to preserve encapsulation.

## 25. Explain constructors.

**Interview-ready answer:**

A constructor initializes a new object and has no return type. If no constructor is declared, Java supplies a default no-argument constructor. `this()` invokes another constructor in the same class and `super()` invokes a parent constructor; either must be the first statement.

## 26. Can a constructor be final, static, abstract, or overridden?

**Interview-ready answer:**

No. Constructors are not inherited and therefore cannot be overridden. They create instances, so static and abstract do not apply, and final is unnecessary.

## 27. What is a static block?

**Interview-ready answer:**

A static initialization block runs once when the class is initialized. It is used for complex static initialization, although simple field initializers or dependency-injection configuration are usually clearer.

## 28. Checked versus unchecked exceptions

**Interview-ready answer:**

Checked exceptions must be caught or declared and are suitable when a caller can reasonably recover, such as an I/O failure. Unchecked exceptions extend `RuntimeException` and usually indicate programming errors, invalid state, or business validation failures. I avoid forcing checked exceptions through every layer when callers cannot meaningfully recover.

## 29. `throw` versus `throws`

**Interview-ready answer:**

`throw` actually raises one exception object inside code. `throws` declares possible exceptions in a method signature. A method can declare multiple exceptions but throws one object at a time.

## 30. What is try-with-resources?

**Interview-ready answer:**

Try-with-resources automatically closes resources that implement `AutoCloseable`, even when an exception occurs. Resources close in reverse declaration order. It is safer and clearer than manual closing in `finally`.

```java
try (BufferedReader reader = Files.newBufferedReader(path)) {
    return reader.readLine();
}
```

## 31. What happens if both `try` and `finally` throw exceptions?

**Interview-ready answer:**

With a normal `finally`, the exception from `finally` can hide the original exception, which is a reason not to throw from cleanup code. Try-with-resources preserves the primary exception and records close failures as suppressed exceptions.

## 32. Can `finally` fail to execute?

**Interview-ready answer:**

Yes, in exceptional cases such as `System.exit()`, JVM termination, process kill, power failure, or an infinite loop before control reaches it. A return statement does not normally skip `finally`.

## 33. What is serialization?

**Interview-ready answer:**

Java serialization converts an object graph into bytes and reconstructs it later. `serialVersionUID` identifies the serialized class version. Native Java serialization has security and compatibility concerns, so for service communication I prefer explicit formats such as JSON, Avro, or Protobuf.

## 34. `transient` and `volatile`

**Interview-ready answer:**

`transient` excludes a field from default Java serialization. `volatile` is a concurrency keyword that provides visibility and ordering guarantees for reads and writes. They solve unrelated problems.

## 35. Shallow copy versus deep copy

**Interview-ready answer:**

A shallow copy creates a new outer object but shares nested object references. A deep copy recursively copies mutable nested state. I prefer copy constructors or explicit mapping because `Cloneable` has awkward semantics and commonly produces shallow copies.

## 36. What is reflection?

**Interview-ready answer:**

Reflection inspects or invokes classes, fields, constructors, and methods at runtime. Frameworks use it for dependency injection, mapping, and testing. It should be used carefully because it can reduce type safety, affect performance, and bypass encapsulation.

## 37. What are annotations?

**Interview-ready answer:**

Annotations attach metadata to code. Their retention can be source-only, stored in class files, or available at runtime. Spring reads runtime annotations such as `@Service` and `@Transactional` to apply framework behavior, often through reflection and proxies.

## 38. What is the class-loading order?

**Interview-ready answer:**

At a high level, a class is loaded, linked, and initialized. Linking includes verification, preparation, and resolution. During initialization, static fields and static blocks run in source order, with the parent class initialized before the child.

## Common traps

- Saying Java is pass-by-reference
- Comparing wrappers or strings with `==`
- Overriding `equals()` without `hashCode()`
- Returning mutable internal collections from an immutable class
- Catching `Exception` everywhere and hiding failures
- Using exceptions for normal control flow
- Claiming `finally` always runs without qualification

