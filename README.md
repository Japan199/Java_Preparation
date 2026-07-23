# Java Developer Interview Handbook

This repository is a practical interview-preparation guide for a **Java Senior Developer – Java, Spring Boot, and Microservices** role, tailored to approximately **4 years 8 months of experience**.

The answers are intentionally concise and conversational. They are written so that you can say them directly in an interview, then expand with examples from your project.

> Interview questions vary by panel and project. This handbook covers the commonly tested concepts and the depth normally expected from an experienced Java developer; it is not an official Infosys question bank.

## Study order

| Phase | Topics | Files |
|---|---|---|
| 1. Core foundation | Java, OOP, strings, exceptions, generics | [01-java-fundamentals.md](01-java-fundamentals.md) |
| 2. Daily Java | Collections, Java 8+, streams, functional programming | [02-collections-generics-java8.md](02-collections-generics-java8.md) |
| 3. Advanced Java | Threads, concurrency, JVM, memory, GC | [03-concurrency-jvm.md](03-concurrency-jvm.md) |
| 4. Spring foundation | Spring Core, DI, AOP, MVC | [04-spring-core-mvc.md](04-spring-core-mvc.md) |
| 5. Spring Boot | Auto-configuration, configuration, Actuator, production readiness | [05-spring-boot.md](05-spring-boot.md) |
| 6. Persistence | SQL, JPA, Hibernate, transactions | [06-jpa-hibernate-sql.md](06-jpa-hibernate-sql.md) |
| 7. APIs and security | REST, validation, exception handling, Spring Security, JWT | [07-rest-security.md](07-rest-security.md) |
| 8. Microservices | Patterns, communication, discovery, gateway, resilience | [08-microservices.md](08-microservices.md) |
| 9. Distributed systems | Kafka, caching, observability, Docker, Kubernetes, CI/CD | [09-distributed-systems-devops.md](09-distributed-systems-devops.md) |
| 10. Quality | JUnit, Mockito, integration tests, clean code, design patterns | [10-testing-design-patterns.md](10-testing-design-patterns.md) |
| 11. Practical rounds | Coding, SQL, debugging, scenario questions | [11-coding-sql-scenarios.md](11-coding-sql-scenarios.md) |
| 12. Senior-level round | System design, project explanation, managerial and HR answers | [12-system-design-project-hr.md](12-system-design-project-hr.md) |
| Final revision | Rapid-fire questions and checklists | [13-rapid-revision.md](13-rapid-revision.md) |

## How to answer as an experienced developer

Use this four-part structure for scenario and project questions:

1. **State the concept or decision.**
2. **Explain why it was suitable.**
3. **Give one real project example.**
4. **Mention the trade-off or measurable result.**

Example:

> “We used a circuit breaker on the payment-service call so that repeated downstream failures would not exhaust our request threads. We configured Resilience4j to open the circuit after the failure-rate threshold, returned a controlled fallback, and monitored the circuit state through Actuator. This protected the order API during payment outages, although the fallback had to be designed carefully to avoid hiding business failures.”

## Four-week preparation plan

### Week 1: Java depth

- Days 1–2: Java fundamentals, OOP, strings, exceptions
- Days 3–4: Collections, generics, streams, Java 8+
- Days 5–6: Concurrency, JVM, GC
- Day 7: Coding practice and revision

### Week 2: Spring and persistence

- Days 1–2: Spring Core, AOP, MVC
- Days 3–4: Spring Boot, configuration, Actuator
- Days 5–6: SQL, JPA, Hibernate, transactions
- Day 7: Build and explain one CRUD API

### Week 3: Microservices

- Days 1–2: Service decomposition, communication, gateway, discovery
- Days 3–4: Resilience, distributed transactions, Kafka
- Days 5–6: Security, caching, observability, Docker/Kubernetes
- Day 7: Design one end-to-end system

### Week 4: Interview simulation

- Practice two coding and two SQL problems daily.
- Explain your current project in two, five, and ten minutes.
- Rehearse production-incident, optimization, conflict, and ownership stories.
- Run three mock interviews: Java, Spring/microservices, and managerial.
- Use the rapid-revision file on the final two days.

## Personalization checklist

Replace generic examples with facts from your experience:

- Services you owned and their business responsibilities
- Java and Spring Boot versions
- Database, messaging system, cache, and cloud platform
- Traffic volume, latency, error rate, or batch size
- One performance improvement with numbers
- One production incident and its root cause
- One difficult technical decision and its trade-offs
- Testing strategy and code-review responsibilities
- Deployment and monitoring workflow

Never claim a tool or pattern you have not used. It is stronger to say, “I understand it and would evaluate it this way,” than to invent experience.

