# Rapid Revision and Final Interview Checklist

## 60 Java one-liners

1. Java is pass-by-value; object-reference values are copied.
2. `==` compares object identity; `equals()` compares logical equality when overridden.
3. Equal objects must have equal hash codes.
4. String is immutable and safely shareable.
5. `StringBuilder` is preferred for single-threaded repeated concatenation.
6. A final reference does not make the object immutable.
7. Composition usually creates less coupling than inheritance.
8. Checked exceptions must be declared or caught; unchecked exceptions do not.
9. Try-with-resources closes `AutoCloseable` resources in reverse order.
10. `ArrayList` is the default general-purpose list.
11. `HashMap` gives average O(1) lookup and allows one null key.
12. Mutable hash-map keys can become unreachable after their hash changes.
13. `ConcurrentHashMap` compound operations should use atomic APIs.
14. `Comparable` is natural ordering; `Comparator` is external ordering.
15. Generics are invariant and mostly implemented through type erasure.
16. PECS means Producer Extends, Consumer Super.
17. A functional interface has one abstract method.
18. Streams are lazy and single-use.
19. `map` transforms; `flatMap` transforms and flattens.
20. `orElse` is eager; `orElseGet` is lazy.
21. Parallel streams use shared resources and require measurement.
22. `volatile` gives visibility and ordering, not compound atomicity.
23. `synchronized` provides mutual exclusion and visibility.
24. `wait()` releases the monitor; `sleep()` does not.
25. Prefer executors over creating a thread per task.
26. Bounded queues prevent invisible memory growth.
27. Deadlocks are reduced by consistent lock order and small critical sections.
28. `CompletableFuture.thenCompose` flattens asynchronous stages.
29. The heap is shared; stacks are per-thread.
30. Reachable but unused objects can still cause Java memory leaks.
31. GC roots are the start of reachability analysis.
32. Metaspace stores class metadata in native memory.
33. JIT optimizes hot code using runtime information.
34. Use JMH for Java microbenchmarks.
35. Virtual threads help high-concurrency blocking I/O, not CPU speed.
36. A Spring singleton is one bean per application context.
37. Constructor injection makes dependencies explicit.
38. `@Repository` also participates in persistence-exception translation.
39. Spring AOP normally works through proxies.
40. Self-invocation can bypass `@Transactional` and `@Async`.
41. `@RestController` includes response-body behavior.
42. Validation belongs on API DTOs; business rules belong deeper.
43. Global advice creates consistent safe errors.
44. Spring Boot auto-configuration is conditional and can back off.
45. Typed `@ConfigurationProperties` is ideal for grouped settings.
46. Readiness controls traffic; liveness controls restart.
47. Actuator endpoints must be selectively exposed and secured.
48. Entities should not be API contracts.
49. The persistence context provides identity tracking and dirty checking.
50. Lazy associations should be fetched intentionally for each use case.
51. N+1 is solved with fetch plans, batches, or projections—not global eager loading.
52. Flush writes pending SQL but is not the same as commit.
53. Optimistic locking detects conflicting versions.
54. `@Transactional` normally rolls back unchecked exceptions by default.
55. Offset pagination degrades at depth; keyset pagination uses a stable cursor.
56. Indexes speed reads but cost storage and writes.
57. Execution plans, not guesses, guide query tuning.
58. Unit tests check isolated behavior; integration tests check boundaries.
59. Testcontainers gives realistic disposable dependencies.
60. Coverage is a signal, not proof of good assertions.

## 40 microservice and production one-liners

1. Microservices trade local simplicity for independent deployment and ownership.
2. Bound services by business capability, not controller/service/repository layers.
3. A modular monolith is often a valid starting point.
4. A service owns its data; other services use APIs or events.
5. Synchronous calls couple latency and availability.
6. Messaging introduces duplicates, ordering issues, and eventual consistency.
7. An API gateway should not become a business-logic monolith.
8. Configure connection and response timeouts for every remote call.
9. Retry only transient, safe, bounded operations with backoff and jitter.
10. A circuit breaker fails fast during sustained downstream failure.
11. A bulkhead isolates scarce resources.
12. Rate limiting protects capacity and fairness.
13. Backpressure is control, not an unbounded buffer.
14. A fallback must be honest about the business outcome.
15. A saga is a sequence of local transactions with compensation.
16. Outbox solves the database/message dual-write gap.
17. Outbox publication can duplicate, so consumers stay idempotent.
18. Eventual consistency must be visible through meaningful states.
19. CAP describes choices during a network partition.
20. CQRS is useful only when read and write needs justify its complexity.
21. Kafka ordering is per partition.
22. Consumer-group parallelism is bounded by partitions.
23. Commit offsets after safely completing the business effect.
24. A dead-letter topic needs monitoring and a replay process.
25. “Exactly once” does not automatically cover external side effects.
26. Cache-aside requires invalidation and stampede handling.
27. TTL limits staleness but does not guarantee consistency.
28. Logs explain events, metrics show trends, traces show request paths.
29. P99 reveals tail latency hidden by averages.
30. Alert on actionable user impact.
31. Containers share the host kernel; VMs include a guest OS.
32. Container images should be minimal, scanned, immutable, and non-root.
33. Kubernetes readiness decides traffic eligibility.
34. Memory limits must leave room beyond Java heap.
35. Rolling deployments require old/new API and schema compatibility.
36. Expand-and-contract enables zero-downtime database changes.
37. Canary deployment limits exposure and uses metrics to promote.
38. The same immutable artifact should move through environments.
39. Feature flags help disable risky behavior but need lifecycle management.
40. Stabilize user impact first, then diagnose and prevent recurrence.

## Answers you must personalize

- Tell me about yourself.
- Explain your project architecture.
- Your exact responsibilities.
- Hardest production issue.
- Performance optimization with before/after numbers.
- A technical disagreement.
- A failure and what changed afterward.
- Why this role and why Infosys?
- Why are you changing?
- Your notice period, location flexibility, and expectations.

## Ten questions to rehearse aloud

1. Explain how `HashMap` works and why key immutability matters.
2. Explain `volatile`, `synchronized`, atomics, and a race condition.
3. Diagnose high CPU or a memory leak.
4. Explain Spring dependency injection and proxy-based AOP.
5. Explain why `@Transactional` sometimes does not work.
6. Fix an N+1 issue without making everything eager.
7. Design a secure, idempotent REST create endpoint.
8. Design a saga using an outbox and idempotent consumers.
9. Handle a failing downstream service without causing a retry storm.
10. Explain one system you personally built using measurable facts.

## Final-day checklist

- Review the job description and map every requirement to one example.
- Prepare a two-minute introduction and project explanation.
- Prepare four STAR stories.
- Review collections, streams, concurrency, transactions, and microservices.
- Solve one Java problem and one SQL window-function problem.
- Confirm your resume dates, project facts, notice period, and availability.
- Prepare two questions for the panel.
- Keep answers to 30–90 seconds unless asked to go deeper.
- Say assumptions before system-design answers.
- If unsure, state what you know and how you would verify it.

## During the interview

- Listen for the exact question; do not answer a different memorized one.
- Lead with a direct answer, then explain.
- Use one relevant project example.
- Mention a trade-off for senior-level questions.
- Think aloud during coding without narrating every keystroke.
- Ask before changing the requirements.
- Never invent production experience.

