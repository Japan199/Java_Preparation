# Kafka, Caching, Observability, Docker, Kubernetes, and CI/CD

## Kafka and messaging

### 1. What are Kafka topics, partitions, and offsets?

**Interview-ready answer:**

A topic is a named event stream split into partitions. A partition is an ordered append-only log. Each record has an offset identifying its position within that partition. Ordering is guaranteed within a partition, not across an entire multi-partition topic.

### 2. How do consumer groups work?

**Interview-ready answer:**

Consumers in the same group divide partitions so one partition is assigned to at most one group member at a time. Different groups consume independently. Maximum active parallelism for a group is limited by partition count.

### 3. How do you choose a message key?

**Interview-ready answer:**

The key controls partitioning and therefore ordering and load distribution. I key by the entity whose event order must be preserved, such as `orderId`, while checking that key distribution will not create hot partitions.

### 4. At-most-once, at-least-once, and exactly-once

**Interview-ready answer:**

At-most-once may lose data but avoids redelivery. At-least-once avoids loss through redelivery but consumers must handle duplicates. Kafka exactly-once features coordinate supported Kafka reads and writes, but end-to-end side effects in databases or external systems still require idempotency and transaction design.

### 5. When do you commit offsets?

**Interview-ready answer:**

I commit only after the business effect is safely completed. Committing first can lose processing on failure; processing first can cause redelivery, so the consumer must be idempotent.

### 6. What is a consumer rebalance?

**Interview-ready answer:**

A rebalance reassigns partitions when group membership or topic partitions change. It can pause processing and cause duplicates if work and offsets are not coordinated. I keep processing bounded, tune polling, handle shutdown correctly, and use cooperative strategies where appropriate.

### 7. Retry topic versus dead-letter topic

**Interview-ready answer:**

Transient failures go through bounded retries with backoff, often using retry topics to avoid blocking a partition. Permanently failing or exhausted messages go to a dead-letter topic with error context. A DLT needs monitoring, ownership, replay controls, and a fix process—it is not a storage graveyard.

### 8. How do you handle schema evolution?

**Interview-ready answer:**

I use a defined schema such as Avro, Protobuf, or governed JSON, enforce backward or forward compatibility as required, add fields with safe defaults, and avoid changing field meaning. Producers and consumers are deployed independently, so both old and new versions must coexist.

### 9. Kafka versus a traditional queue

**Interview-ready answer:**

Kafka is a retained distributed log that supports replay, multiple independent consumer groups, and high-throughput streams. A traditional broker may emphasize per-message routing, priority, or queue semantics. I choose based on delivery model, replay, routing, latency, and operations.

## Caching

### 10. Cache-aside pattern

**Interview-ready answer:**

The application checks the cache, loads from the database on a miss, then populates the cache. On writes it updates the database and invalidates or updates the cache. It is simple, but races, stale values, and cache stampedes need explicit handling.

### 11. Cache eviction strategies

**Interview-ready answer:**

Common policies include LRU, LFU, TTL-based expiration, and size-based eviction. The right policy depends on access patterns and data freshness. TTL is a safety net, not a complete consistency strategy.

### 12. What is a cache stampede?

**Interview-ready answer:**

Many requests miss the same hot key and all query the database simultaneously. I mitigate it using request coalescing or single-flight locking, staggered TTLs, refresh-ahead, bounded concurrency, or serving an acceptable stale value.

### 13. What is cache penetration?

**Interview-ready answer:**

Repeated requests for nonexistent keys bypass the cache and hit the database. I can cache negative results briefly, validate requests, rate-limit abuse, or use a Bloom filter when the dataset and false-positive trade-off fit.

### 14. Local versus distributed cache

**Interview-ready answer:**

A local cache is fast and avoids network calls but each replica has independent, potentially stale state. A distributed cache provides shared state and greater capacity but adds network latency and another dependency. Sometimes a small local near-cache plus distributed cache is appropriate.

## Observability

### 15. Logs, metrics, and traces

**Interview-ready answer:**

Logs describe discrete events, metrics show aggregated trends and support alerts, and traces show one request’s path across components. Together they help answer what failed, how widely, and where time was spent.

### 16. What should you monitor for an API?

**Interview-ready answer:**

I monitor request rate, error rate, latency percentiles, and saturation, plus JVM, thread pool, connection pool, GC, database, cache, and downstream-client metrics. I also monitor business outcomes such as successful orders, not only infrastructure.

### 17. Why percentiles instead of only average latency?

**Interview-ready answer:**

Averages hide slow-tail behavior. P95 or P99 shows what slower users experience and is useful for SLOs. I still interpret percentiles with traffic volume and distribution.

### 18. What are SLI, SLO, and SLA?

**Interview-ready answer:**

An SLI is a measured reliability indicator such as successful-request ratio. An SLO is the internal target for that indicator. An SLA is a formal commitment, often with consequences. An error budget is the allowed unreliability under the SLO.

### 19. How does distributed tracing work?

**Interview-ready answer:**

A trace ID follows an end-to-end request and spans represent individual operations. Context is propagated through HTTP headers or message metadata. OpenTelemetry provides vendor-neutral instrumentation and export. Sampling controls cost but must preserve useful error and high-latency traces.

### 20. What makes a good alert?

**Interview-ready answer:**

A good alert is actionable, tied to user impact or an impending limit, has a clear owner and runbook, and avoids noisy transient signals. I prefer symptom-based alerts on sustained SLO impact, supported by cause-level diagnostics.

## Docker

### 21. Container versus virtual machine

**Interview-ready answer:**

Containers isolate processes while sharing the host kernel, so they start quickly and use fewer resources. VMs virtualize hardware and include a guest OS, providing a stronger isolation boundary at higher cost. Containers are packaging and isolation, not lightweight VMs in every respect.

### 22. How do you build a good Java container image?

**Interview-ready answer:**

I use a trusted minimal runtime image, multi-stage or buildpack-based builds, layer dependencies for caching, run as non-root, avoid secrets in layers, use a read-only filesystem where possible, scan dependencies and images, and set JVM container-aware resource parameters.

### 23. `ENTRYPOINT` versus `CMD`

**Interview-ready answer:**

`ENTRYPOINT` defines the executable and `CMD` supplies default arguments that can be overridden. Exec-form JSON is generally preferred because signals reach the Java process correctly without an extra shell.

## Kubernetes

### 24. Pod, Deployment, and Service

**Interview-ready answer:**

A Pod is the smallest scheduled unit containing one or more containers. A Deployment manages replica rollout and desired state for stateless workloads. A Service provides a stable virtual endpoint and discovery over changing pods.

### 25. ConfigMap versus Secret

**Interview-ready answer:**

A ConfigMap stores non-sensitive configuration. A Secret is intended for sensitive values, but base64 representation alone is not encryption. I also require encryption at rest, RBAC, restricted mounting, rotation, and ideally integration with a secret manager.

### 26. Liveness, readiness, and startup probes

**Interview-ready answer:**

Startup probes protect slow-starting applications from premature liveness checks. Readiness controls traffic eligibility. Liveness detects a process that cannot recover without restart. Incorrect liveness dependencies can create cascading restart loops.

### 27. Resource requests and limits

**Interview-ready answer:**

Requests influence scheduling and guarantee expected capacity; limits cap resource use. CPU limits can throttle and memory-limit breaches can kill the container. I size them from load tests and production metrics and configure JVM heap with room for non-heap and native memory.

### 28. Rolling, blue-green, and canary deployment

**Interview-ready answer:**

Rolling replaces instances gradually with low extra capacity. Blue-green switches traffic between full old and new environments for fast rollback but costs more. Canary sends a small traffic percentage to the new version and expands based on metrics. Database changes must remain compatible during overlap.

### 29. How do you perform zero-downtime database changes?

**Interview-ready answer:**

I use expand-and-contract: add backward-compatible schema first, deploy code that supports both versions, migrate data, switch reads and writes, then remove old schema in a later deployment. I avoid renaming or dropping a column in the same release that stops using it.

## CI/CD

### 30. Describe a production pipeline.

**Interview-ready answer:**

The pipeline builds reproducibly, runs unit and integration tests, performs static analysis and dependency/security scans, packages and signs an immutable artifact, deploys through controlled environments, runs smoke or contract tests, and promotes the same artifact with approval and rollback support. Deployment metrics determine whether rollout continues.

### 31. What is a rollback strategy?

**Interview-ready answer:**

Application rollback restores a previously verified image or routes traffic back. Database changes must be backward-compatible because data rollback is harder. For risky features I use feature flags and canary rollout so disabling or limiting impact is fast.

### 32. Production incident answer structure

> “I first assessed customer impact and stabilized the service using the safest reversible mitigation. I used metrics, traces, logs, and recent-change history to form and test hypotheses. After identifying the root cause, I deployed a verified fix, monitored recovery, documented the timeline, and created prevention actions such as an alert, test, limit, or runbook update. I focus on system improvement rather than individual blame.”

