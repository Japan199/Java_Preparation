# System Design, Project Explanation, Managerial, and HR Round

## System design framework

Use this order:

1. Clarify functional requirements and exclusions.
2. Estimate traffic, storage, payload size, and latency needs.
3. Define APIs and core data model.
4. Draw the high-level request and event flow.
5. Explain data ownership and consistency.
6. Add scaling, caching, messaging, and resilience.
7. Add security, observability, deployment, and failure recovery.
8. State trade-offs and the next bottleneck.

## Design an order management system

### Requirements

- Create and view orders
- Reserve inventory
- Authorize payment
- Track order status
- Notify the customer
- Prevent duplicate orders and overselling

### Interview-ready design

> “I would expose order APIs through an authenticated API gateway. The order service owns order state in a relational database because order transitions and constraints are transactional. Create-order accepts an idempotency key. In the same transaction it stores the order and an outbox event.
>
> A publisher sends the event to Kafka. A saga orchestrates inventory reservation and payment authorization through idempotent commands. On failure, it compensates completed steps, such as releasing inventory. Order status is visible as pending, confirmed, or failed rather than pretending distributed work is immediate.
>
> Read traffic can use a cache where staleness is acceptable. Every remote call has timeouts, bounded retries for safe transient errors, circuit breakers, and correlation-aware tracing. Consumers use event IDs for deduplication, failed messages have bounded retry and dead-letter handling, and reconciliation finds stuck workflows.
>
> I would deploy multiple stateless instances behind a service/load balancer, use database indexes on customer and status queries, partition events by order ID for ordering, and monitor latency, error rate, consumer lag, saga age, payment success, and inventory conflicts. Security includes short-lived tokens, object-level authorization, encryption, secret management, and audit logs.” 

### Likely follow-ups

**How do you prevent duplicate payment?**

Use the order/payment attempt ID as an idempotency key, enforce uniqueness at the payment service, and store the provider result before returning. A retry returns the existing result.

**What if the event is not published after the database commit?**

The transactional outbox stores the event in the same commit. A background publisher retries unpublished records.

**What if the publisher sends twice?**

Consumers deduplicate by event ID or make their state transition naturally idempotent.

**How do you handle events arriving out of order?**

Partition related events by aggregate ID, include sequence/version information, reject or defer stale transitions, and reconcile gaps.

**How do you scale the database?**

First optimize queries and indexes, cache safe reads, use replicas for suitable reads, archive old data, and partition when volume justifies it. Sharding is a later step because it complicates transactions and queries.

## Explain your project

### Two-minute template

> “My project is a [domain] platform used by [users]. Its main business goal is [goal]. We use Java [version], Spring Boot, [database], [messaging/cache], and deploy through [platform].
>
> I am responsible for [two or three owned services/features]. My work includes requirement analysis, API and database design, implementation, code review, automated testing, release support, and production troubleshooting.
>
> One important problem I solved was [problem]. The cause was [root cause]. I changed [technical action], validated it using [test/metrics], and improved [latency/error rate/cost] from [before] to [after].
>
> The architecture uses [communication approach], [consistency strategy], and [resilience controls]. We monitor it using [metrics/logs/traces]. My main learning was [specific engineering learning].”

Fill every bracket with truthful, measurable detail.

### Architecture questions to prepare

- What business capability does each service own?
- Why did you select microservices?
- How do services communicate?
- How do you maintain data consistency?
- How do you secure APIs?
- How do you deploy and roll back?
- What happens when a dependency fails?
- How do you trace a request?
- What is your traffic, data volume, and latency?
- Which design decision would you change now?

## STAR stories to prepare

STAR means Situation, Task, Action, Result. Keep Situation and Task short; spend most time on your actions and measurable result.

### Production incident

> “A release caused order API P99 latency to cross our target. I owned the investigation. Traces showed a new mapping path triggering N+1 queries. I mitigated impact by rolling back, replaced the access with a projection and fetch plan, added a query-count integration test, and validated it under production-like load. P99 returned from [X] to [Y], and we added a release dashboard to catch similar regressions.”

### Performance improvement

> “The batch took [X] minutes and missed its SLA. Profiling showed repeated database calls and unbounded in-memory processing. I changed it to chunked reads, batch writes, and bounded parallelism, then measured database load and failure recovery. Runtime fell to [Y] with stable memory. I documented the tuning limits so future changes would not overload the database.”

### Disagreement

> “We disagreed on synchronous versus event-driven processing. I wrote down latency, consistency, failure, and operational requirements and built a small comparison using our traffic. We agreed the critical validation remained synchronous while notifications became event-driven. That met the response target without adding eventual consistency to the core invariant.”

### Ownership

> “A recurring failure did not have a clear team owner. I collected incidents, identified the shared failure mode, proposed a bounded retry and reconciliation process, coordinated owners, and added a dashboard and runbook. The repeat incident rate dropped by [measured result].”

## Senior developer questions

### 1. How do you make technical decisions?

**Interview-ready answer:**

I start from functional and quality requirements, constraints, and likely failure modes. I compare the simplest viable options using evidence such as a small proof, load test, or operational history, document trade-offs, involve affected engineers, and define how the decision will be measured and revisited.

### 2. How do you handle code reviews?

**Interview-ready answer:**

I prioritize correctness, security, failure behavior, compatibility, data access, tests, and observability. I make feedback specific and respectful, explain impact, and separate required changes from suggestions. For major design concerns I discuss directly rather than creating a long comment thread.

### 3. How do you mentor junior developers?

**Interview-ready answer:**

I explain the reasoning behind decisions, give scoped ownership, pair on difficult work, and use review feedback as teaching rather than simply rewriting code. I gradually reduce support as confidence grows and make standards accessible through examples and documentation.

### 4. How do you handle tight deadlines?

**Interview-ready answer:**

I clarify the non-negotiable outcome, split scope into safe increments, identify technical and dependency risks early, and make trade-offs visible to stakeholders. I do not silently remove testing, security, or rollback capability; I negotiate lower-priority scope and track intentional debt.

### 5. How do you estimate work?

**Interview-ready answer:**

I decompose work into understood pieces, list assumptions and dependencies, include testing, review, deployment, and migration effort, and use ranges when uncertainty is high. I reduce uncertainty with a short spike and update estimates when facts change.

### 6. Tell me about a failure.

**Interview-ready answer:**

I choose a real technical or coordination mistake, state my responsibility without excuses, explain the impact and immediate correction, and focus on a concrete change that prevents recurrence. The answer should demonstrate learning, not claim that the failure was secretly a success.

### 7. How do you ensure quality?

**Interview-ready answer:**

I combine clear acceptance criteria, small reviewed changes, automated unit and integration tests, static and security checks, production-like validation, observable rollout, and post-release monitoring. Quality is built throughout delivery rather than added during a final testing phase.

## HR questions

### Tell me about yourself.

> “I am a Java backend developer with 4 years and 8 months of experience building and supporting APIs and distributed services. My main skills are Java, Spring Boot, REST APIs, SQL/JPA, and microservice patterns, along with testing and production troubleshooting. In my current role I own [service/feature], participate from design through deployment, and have delivered [one measurable result]. I am now looking for a senior developer role where I can take broader technical ownership and contribute to reliable enterprise systems.”

### Why Infosys?

> “I am interested in Infosys because the role combines the technologies I have been working with—Java, Spring Boot, and microservices—with large enterprise delivery. I want exposure to complex business systems where engineering quality, client communication, and production ownership matter. I can contribute hands-on backend experience while growing into broader design and technical-lead responsibilities.”

Avoid vague claims; connect the answer to the actual job description and what you can contribute.

### Why are you changing jobs?

> “I have learned a lot in my current role and completed meaningful backend work. I am now looking for broader ownership, more complex distributed-system challenges, and a path to senior engineering responsibilities. I am making the move for growth and role alignment, not because of a negative relationship with my current employer.”

### What are your strengths?

> “My main strength is structured problem-solving. I trace an issue from user impact through logs, metrics, code, and data rather than guessing. I also take ownership through testing, deployment, and monitoring. For example, [brief measurable example].”

### What is your weakness?

> “Earlier I sometimes spent too long refining a design before sharing it. I now time-box investigation, share an early proposal with assumptions, and get feedback from affected teammates. That has reduced rework while keeping the design quality strong.”

### Where do you see yourself in three years?

> “I want to be a strong senior engineer who can own a service or workstream end to end, guide design decisions, mentor developers, and remain hands-on with implementation and production reliability. I am especially interested in deepening distributed-system and cloud architecture skills.”

### Expected salary

> “I am looking for compensation aligned with the role’s responsibilities, my 4 years 8 months of relevant experience, and the overall package. I am open to discussing the approved range for this position after we confirm the role and expectations are a good match.”

### Do you have questions for us?

Ask two or three:

- What would success in the first six months look like?
- What architecture and scale would this role work with?
- How much ownership does the team have from design through production?
- What are the team’s biggest current technical challenges?
- How are code quality, releases, and on-call support handled?

## Authenticity rule

Do not memorize invented metrics or architectures. Replace every example with your own project facts. If you know a concept but have not used it, say:

> “I have not implemented that in production yet. My understanding is [concise explanation]. In this system I would evaluate it based on [relevant trade-offs].”

