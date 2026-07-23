# Testing, Clean Code, SOLID, and Design Patterns

## Testing

### 1. Unit, integration, and end-to-end tests

**Interview-ready answer:**

Unit tests verify one unit with fast controlled dependencies. Integration tests verify real boundaries such as database mapping, Spring configuration, or messaging. End-to-end tests verify complete user flows but are slower and more fragile. I use many focused unit tests, meaningful integration tests, and a smaller number of critical end-to-end tests.

### 2. What makes a good unit test?

**Interview-ready answer:**

It is deterministic, fast, isolated, readable, and verifies behavior rather than private implementation. I structure it as arrange, act, assert and give it a name describing the condition and outcome.

### 3. Mock versus stub versus spy

**Interview-ready answer:**

A stub returns prepared data. A mock also verifies interactions. A spy wraps a real object and allows selective stubbing or verification. I mock external collaborators at the unit boundary and avoid mocking value objects or the class under test.

### 4. Common Mockito annotations

**Interview-ready answer:**

`@Mock` creates a mock, `@Spy` wraps a real instance, `@InjectMocks` attempts to construct and inject mocks, and `@Captor` captures arguments. I still prefer explicit constructor creation when it makes test dependencies clearer.

### 5. `when(...).thenReturn(...)` versus `doReturn(...).when(...)`

**Interview-ready answer:**

The normal style is `when(mock.call()).thenReturn(value)`. For spies, calling a real method during stubbing may be unsafe, so `doReturn(value).when(spy).call()` avoids invoking it. Heavy spy use often indicates a design that should be simplified.

### 6. What should you verify?

**Interview-ready answer:**

I verify observable results first. I verify interactions only when the interaction is itself part of the behavior, such as publishing an event once or not charging on validation failure. Over-verifying call order makes refactoring difficult.

### 7. `@SpringBootTest` versus slice tests

**Interview-ready answer:**

`@SpringBootTest` loads most or all of the application and is useful for broad integration. Slice tests such as MVC or JPA slices load a focused part and are faster. I select the smallest context that proves the behavior.

### 8. What is MockMvc?

**Interview-ready answer:**

MockMvc tests the Spring MVC layer without starting a real server. It verifies routing, validation, serialization, status codes, and exception handling. A full HTTP integration test is still useful for selected end-to-end configuration.

### 9. Why use Testcontainers?

**Interview-ready answer:**

Testcontainers starts disposable real dependencies such as PostgreSQL, Kafka, or Redis for tests. It catches dialect, transaction, and protocol behavior that in-memory replacements may miss while keeping environments reproducible.

### 10. What is a contract test?

**Interview-ready answer:**

A contract test verifies that a provider and consumer agree on request, response, or event schemas and semantics. Consumer-driven contracts help services deploy independently, but they supplement rather than replace integration and behavior tests.

### 11. How do you test asynchronous processing?

**Interview-ready answer:**

I use a real or realistic broker in integration tests, publish an input, and wait with a bounded polling assertion for the expected outcome. I verify duplicates, retries, malformed events, and idempotency without using arbitrary fixed sleeps.

### 12. How do you test a transactional service?

**Interview-ready answer:**

Unit tests verify business decisions and collaborator calls. An integration test with the actual database verifies commit, rollback, constraints, locking, and ORM behavior. I ensure the test framework’s own transaction does not accidentally hide the production boundary.

### 13. Code coverage

**Interview-ready answer:**

Coverage shows which code executed, not whether behavior was correctly tested. I use it to find gaps, not as the sole quality target. Risky business rules, failures, and boundaries deserve stronger assertions than trivial getters.

## SOLID and clean design

### 14. Explain SOLID.

**Interview-ready answer:**

- **Single Responsibility:** a unit has one cohesive reason to change.
- **Open/Closed:** extend behavior without repeatedly modifying stable code.
- **Liskov Substitution:** subtypes honor the parent contract.
- **Interface Segregation:** clients depend on small relevant interfaces.
- **Dependency Inversion:** high-level policy depends on abstractions, not infrastructure details.

I use these as design guidance, not rules that require an interface for every class.

### 15. Example of Single Responsibility

**Interview-ready answer:**

An `OrderService` should coordinate order rules, not also generate PDFs, send email, and construct SQL. Those responsibilities change for different reasons and should be delegated to focused collaborators.

### 16. Explain Liskov Substitution with an example.

**Interview-ready answer:**

If a subtype cannot honor the behavior promised by its base type, substitution is broken. A classic example is a read-only collection subtype whose inherited `add()` unexpectedly throws. I model capabilities through accurate interfaces instead of forcing an invalid hierarchy.

### 17. What is dependency inversion in Spring?

**Interview-ready answer:**

The application service depends on a domain-facing interface such as `PaymentGateway`, while an HTTP adapter implements it. Spring injects the implementation. Business logic can then be tested without network code and infrastructure can change independently.

### 18. DRY, KISS, and YAGNI

**Interview-ready answer:**

DRY avoids duplicated knowledge, KISS favors the simplest design that works, and YAGNI avoids speculative features. I do not remove all similar-looking code prematurely; two pieces may evolve differently, so I abstract after understanding the stable common concept.

## Design patterns

### 19. Strategy pattern

**Interview-ready answer:**

Strategy encapsulates interchangeable algorithms behind a common contract. For example, payment methods implement `PaymentStrategy`, and a factory or registry selects one. It removes growing conditionals and supports focused testing.

### 20. Factory pattern

**Interview-ready answer:**

A factory centralizes object creation or implementation selection so callers depend on the product contract. In Spring, the container is already a factory, but an application-level factory is still useful when selection depends on runtime business data.

### 21. Builder pattern

**Interview-ready answer:**

Builder constructs complex objects step by step with readable named methods, especially when there are many optional parameters. It can validate before creation and avoid telescoping constructors. Records or constructors are simpler for small fixed data objects.

### 22. Singleton pattern and Spring singleton scope

**Interview-ready answer:**

The classic singleton enforces one instance through code. Spring singleton scope means one bean instance per application context and lets the container manage it. I prefer container-managed lifecycle and keep singleton services stateless.

### 23. Template Method pattern

**Interview-ready answer:**

Template Method defines an algorithm skeleton in a base class while subclasses customize steps. It is useful for stable related workflows but relies on inheritance. Strategy or composition is often more flexible when behavior must vary independently.

### 24. Observer pattern

**Interview-ready answer:**

Observers subscribe to events from a subject. It decouples the producer from multiple reactions but introduces ordering, error-handling, and lifecycle concerns. Spring application events work within a process; a message broker serves distributed consumers.

### 25. Adapter pattern

**Interview-ready answer:**

Adapter translates one interface or model into another. I use it to isolate external payment APIs or legacy systems so the domain does not depend on vendor-specific types.

### 26. Decorator pattern

**Interview-ready answer:**

Decorator wraps an object with the same contract to add behavior such as caching, metrics, or retry. It composes behavior dynamically without changing the original class. Spring proxies are conceptually similar for cross-cutting behavior.

### 27. Proxy pattern

**Interview-ready answer:**

A proxy controls access to a target, potentially adding lazy loading, security, transactions, or remote access. Spring AOP and Hibernate lazy associations use proxies, which is why method boundaries and object identity behavior matter.

### 28. Chain of Responsibility

**Interview-ready answer:**

A request passes through a sequence of handlers, each deciding whether to process or continue. Servlet filters and Spring Security’s filter chain are practical examples. Ordering and clear ownership are important.

### 29. Facade pattern

**Interview-ready answer:**

A facade exposes a simpler high-level interface over several components. An application service can act as a facade for a use case, but it should not become an unstructured god class.

### 30. Repository pattern

**Interview-ready answer:**

A repository presents collection-like access to domain aggregates and hides persistence mechanics. Spring Data generates much of the implementation, but query design and aggregate boundaries still need deliberate decisions.

## Code review answer

> “I first check correctness, business edge cases, security, transaction boundaries, concurrency, and failure behavior. Then I review API compatibility, query count, resource limits, tests, observability, and readability. I keep feedback specific and explain impact, distinguish blocking issues from suggestions, and use automated formatting and static analysis for mechanical rules.”

