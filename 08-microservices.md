# Microservices: Fundamentals, Patterns, and Resilience

## Foundations

### 1. What are microservices?

**Interview-ready answer:**

Microservices are independently deployable services aligned to business capabilities, with clear ownership of data and behavior. Their benefit is independent evolution and scaling, but they add network, data-consistency, deployment, and observability complexity. I do not choose them solely because an application is large.

### 2. Monolith versus microservices

**Interview-ready answer:**

A monolith has simpler local calls, transactions, testing, and deployment, but can become hard to change and scale organizationally if poorly modularized. Microservices allow independent deployment and scaling but introduce distributed-system failure modes. A modular monolith is often a good starting point until service boundaries and operational needs justify distribution.

### 3. How do you identify service boundaries?

**Interview-ready answer:**

I use business capabilities and domain boundaries rather than technical layers. A service should own cohesive behavior and data, have a clear team owner, and change for related reasons. Domain-driven design bounded contexts and event-storming help reveal those boundaries.

### 4. Why should each service own its database?

**Interview-ready answer:**

Database ownership preserves autonomy and prevents other services from coupling to internal schemas. Other services access data through APIs or events. This introduces duplication and eventual consistency, but shared tables make independent change and deployment difficult.

### 5. Synchronous versus asynchronous communication

**Interview-ready answer:**

Synchronous HTTP or gRPC is simple when an immediate response is required but couples availability and latency. Messaging decouples time and supports buffering and fan-out but adds eventual consistency, duplicate handling, and operational complexity. I use each based on the business workflow, not one universally.

### 6. REST versus gRPC

**Interview-ready answer:**

REST/JSON is widely interoperable and easy for public APIs. gRPC uses strongly typed Protobuf contracts, efficient binary transport, and supports streaming, making it useful for controlled internal communication. Browser and debugging needs, contract governance, latency, and ecosystem determine the choice.

### 7. What is service discovery?

**Interview-ready answer:**

Service discovery maps a logical service name to healthy instances. Client-side discovery lets clients select an instance; server-side discovery routes through a load balancer. Kubernetes commonly provides service discovery through DNS and Services, reducing the need for an application-level registry.

### 8. What is an API gateway?

**Interview-ready answer:**

An API gateway is a controlled entry point that routes requests and may handle authentication, TLS termination, rate limiting, request shaping, and observability. I keep business logic out of it so it does not become a new monolith or bottleneck.

### 9. Client-side versus server-side load balancing

**Interview-ready answer:**

With client-side load balancing, the caller chooses an instance using discovery information. With server-side load balancing, a proxy or platform service chooses. Server-side approaches simplify clients; client-side approaches can provide direct control but require consistent client behavior.

### 10. What is centralized configuration?

**Interview-ready answer:**

Centralized configuration manages environment-specific values outside service artifacts. It needs access control, versioning, validation, auditability, and safe refresh behavior. Secrets belong in a dedicated secret-management mechanism rather than normal config.

## Resilience

### 11. What is a timeout?

**Interview-ready answer:**

A timeout limits how long a call can consume resources. I configure connection, read, and overall deadlines based on the upstream latency budget. Without timeouts, slow dependencies can exhaust threads and connections.

### 12. What is a retry?

**Interview-ready answer:**

A retry repeats a failed operation when the failure is likely transient. I use a small bounded count, exponential backoff with jitter, and only retry idempotent or deduplicated operations. I do not retry validation failures or overload blindly because retries can amplify an incident.

### 13. What is a circuit breaker?

**Interview-ready answer:**

A circuit breaker observes calls and opens when failures or slow calls exceed a threshold. While open, it fails fast; after a wait it permits trial calls in half-open state. It protects resources and allows recovery, but a fallback must preserve correct business meaning.

### 14. Circuit breaker versus retry

**Interview-ready answer:**

Retry handles an individual transient failure by trying again. A circuit breaker stops repeated calls when a dependency is broadly unhealthy. They can work together, but retries should be inside a carefully designed resilience policy so they do not inflate the circuit’s traffic.

### 15. What is a bulkhead?

**Interview-ready answer:**

A bulkhead isolates resources so failure in one dependency or workload does not consume all threads or connections. I can use separate pools, semaphores, queues, or service instances. The limits must align with downstream capacity.

### 16. What is rate limiting?

**Interview-ready answer:**

Rate limiting controls requests per client, tenant, or operation using algorithms such as token bucket or sliding window. It protects capacity and fairness and should return a clear `429` response with retry guidance where appropriate.

### 17. What is backpressure?

**Interview-ready answer:**

Backpressure prevents producers from overwhelming consumers. It can use bounded queues, demand signaling, rate controls, or rejection. Buffering alone is not a complete solution because an unbounded buffer only delays failure.

### 18. What is graceful degradation?

**Interview-ready answer:**

Graceful degradation returns a reduced but honest experience when a noncritical dependency fails, such as omitting recommendations while checkout remains available. I do not return fake success for critical operations.

## Data consistency

### 19. What is the Saga pattern?

**Interview-ready answer:**

A saga coordinates a business transaction as local transactions across services. If a later step fails, compensating actions undo or offset earlier work. Choreography uses events; orchestration uses a coordinator. Compensation is business logic and may not perfectly restore the past.

### 20. Saga choreography versus orchestration

**Interview-ready answer:**

Choreography is decentralized and services react to events, which reduces a central dependency but makes long workflows harder to visualize. Orchestration makes the workflow explicit in a coordinator but centralizes coordination logic. I choose based on workflow complexity and operational clarity.

### 21. What is the transactional outbox pattern?

**Interview-ready answer:**

The service writes its business change and an outbox event in the same local database transaction. A separate publisher sends outbox records to the broker. This avoids the dual-write gap, although publication may repeat, so consumers must be idempotent.

### 22. What is idempotent consumption?

**Interview-ready answer:**

An idempotent consumer can process the same message more than once without duplicating the business effect. I use a unique event ID with a processed-message record, a naturally idempotent state transition, or an atomic database constraint.

### 23. What is eventual consistency?

**Interview-ready answer:**

Eventual consistency means replicas or services may temporarily disagree but converge when updates propagate. I make this visible in product behavior through statuses such as `PROCESSING`, design retries and reconciliation, and avoid promising immediate consistency the architecture cannot provide.

### 24. What is CQRS?

**Interview-ready answer:**

CQRS separates write models from read models when their requirements differ significantly. It can improve scaling and domain modeling but introduces synchronization and operational complexity. I use it for a specific need, not automatically in every microservice.

### 25. What is event sourcing?

**Interview-ready answer:**

Event sourcing stores immutable domain events as the source of truth and rebuilds state by replaying them. It provides auditability and temporal reconstruction but adds event-versioning, replay, storage, and debugging complexity. It is different from simply publishing integration events.

### 26. CAP theorem

**Interview-ready answer:**

During a network partition, a distributed system must trade off consistent responses against availability. CAP is about behavior under partitions, not a permanent choice of only two properties. The decision may differ per operation; for example, payment authorization may prefer rejecting uncertainty while a product catalog may serve slightly stale data.

### 27. Strong consistency versus eventual consistency

**Interview-ready answer:**

Strong consistency makes reads observe the required latest ordering but may need coordination and reduce availability or latency. Eventual consistency improves autonomy and availability but exposes temporary staleness. I select it per business invariant.

## Architecture patterns

### 28. Strangler pattern

**Interview-ready answer:**

The strangler pattern incrementally replaces a legacy system by routing selected capabilities to new services while the old system continues handling the rest. It reduces big-bang migration risk and needs clear routing, data ownership, and retirement criteria.

### 29. Anti-corruption layer

**Interview-ready answer:**

An anti-corruption layer translates an external or legacy model into the service’s own domain model. It prevents foreign concepts and instability from spreading through the codebase.

### 30. Sidecar pattern

**Interview-ready answer:**

A sidecar runs beside the application instance and provides infrastructure capabilities such as proxying, certificate handling, or telemetry. It separates concerns but adds resource use and operational components.

### 31. How do you handle API compatibility?

**Interview-ready answer:**

I prefer additive changes, tolerate unknown fields, avoid changing existing field meaning, and use consumer-driven contract tests. Breaking changes require a version and migration window. For events, schemas also need compatibility rules and version-aware consumers.

### 32. What is a correlation ID?

**Interview-ready answer:**

A correlation ID links logs for one business request across services. Distributed tracing goes further by recording trace and span relationships. I propagate approved context through HTTP headers or message metadata and avoid using identifiers that expose sensitive data.

## End-to-end order workflow answer

> “The order service validates the request and writes the order plus an outbox event in one transaction. The event is published to the broker. Inventory reserves stock idempotently, and payment authorizes payment. A saga coordinator tracks outcomes. If payment fails, inventory receives a compensation command to release the reservation and the order becomes failed. Every message has an event ID and correlation ID, consumers retry transient failures with backoff, poison messages go to a dead-letter flow, and reconciliation detects stuck orders. This gives reliable eventual consistency without a distributed database transaction.”

## Common traps

- Saying microservices are always better than a monolith
- Using one shared database for convenience without acknowledging coupling
- Retrying every error
- Calling a fallback “success” when the business operation failed
- Claiming a broker delivers exactly once end to end
- Treating compensation as a technical database rollback
- Ignoring duplicate and out-of-order events

