# SQL, JPA, Hibernate, and Transactions

## JPA and Hibernate

### 1. JPA versus Hibernate

**Interview-ready answer:**

JPA, now Jakarta Persistence, is a specification defining ORM contracts and annotations. Hibernate is a popular implementation with additional features. I usually program to JPA APIs while understanding Hibernate behavior because performance issues depend on the provider.

### 2. What are entity lifecycle states?

**Interview-ready answer:**

An entity can be transient, managed, detached, or removed. Managed entities are tracked in the persistence context, so changes are detected and written during flush. Detached entities are no longer tracked; `merge()` copies their state into a managed instance and returns that instance.

### 3. What is the persistence context?

**Interview-ready answer:**

The persistence context is a unit-of-work and first-level cache that tracks managed entities by identity. Within it, loading the same entity ID returns the same managed instance. It enables dirty checking and coordinated writes.

### 4. What is dirty checking?

**Interview-ready answer:**

Hibernate tracks managed entity state. At flush time, it detects changes and generates updates, so an explicit save is not required for every modification inside a transaction. I keep transaction boundaries clear to avoid unexpected writes.

### 5. `persist()` versus `merge()`

**Interview-ready answer:**

`persist()` makes a new entity managed. `merge()` copies state from a detached or transient object into a managed instance and returns it; the supplied object itself does not become managed. For updates, I prefer loading the entity and changing allowed fields to prevent accidental overwrites.

### 6. `flush()` versus commit

**Interview-ready answer:**

Flush synchronizes pending persistence-context changes to the database, but the transaction may still roll back. Commit completes the transaction and normally triggers a flush. Queries can also trigger auto-flush to preserve consistency.

### 7. Lazy versus eager fetching

**Interview-ready answer:**

Lazy fetching loads an association when accessed; eager fetching requests it immediately but does not guarantee one efficient query. I generally keep collections lazy and fetch exactly what a use case needs using projections, fetch joins, or entity graphs.

### 8. What is the N+1 query problem?

**Interview-ready answer:**

One query loads N parent rows, then accessing a lazy association triggers one additional query per parent. I detect it using SQL logs or query metrics and fix it with a fetch join, entity graph, batch fetching, or a purpose-built projection. Making everything eager can create larger problems.

### 9. What is `LazyInitializationException`?

**Interview-ready answer:**

It occurs when code accesses an unloaded lazy association after its persistence context is closed. I solve it by fetching required data within a service transaction and mapping to a DTO, not by enabling long-lived sessions or changing every relationship to eager.

### 10. Why avoid exposing entities directly from controllers?

**Interview-ready answer:**

Entities represent persistence state, not an API contract. Exposing them can cause lazy loading, cyclic serialization, over-posting, leaking internal fields, and breaking clients when the schema changes. DTOs provide a stable boundary and deliberate data selection.

### 11. Cascade types versus `orphanRemoval`

**Interview-ready answer:**

Cascades propagate entity operations such as persist or remove from parent to child. `orphanRemoval=true` deletes a child removed from the parent relationship. I use both only when the child lifecycle is genuinely owned by the aggregate.

### 12. Owning side of a relationship

**Interview-ready answer:**

The owning side controls the foreign-key update; in a bidirectional relationship, `mappedBy` marks the inverse side. I update both in-memory sides through helper methods so the object graph stays consistent, even though only the owning side drives persistence.

### 13. Why can Lombok-generated `equals()` be risky on entities?

**Interview-ready answer:**

Including generated IDs, lazy associations, or mutable fields can break hash collections, trigger queries, or cause recursion. Entity identity changes before and after persistence, so I design equality deliberately, often using a stable natural key where available.

### 14. What is optimistic locking?

**Interview-ready answer:**

Optimistic locking uses a `@Version` field. An update includes the expected version, and if another transaction already changed the row, the update fails instead of silently overwriting data. It works well when conflicts are uncommon and can be retried or presented to the user.

### 15. What is pessimistic locking?

**Interview-ready answer:**

Pessimistic locking asks the database to lock rows during a transaction. It is useful for short, high-contention critical sections but can reduce throughput and cause blocking or deadlocks. I use timeouts and consistent access order.

### 16. First-level versus second-level cache

**Interview-ready answer:**

The first-level cache belongs to a persistence context and is mandatory. The second-level cache is optional and shared across sessions for suitable entity data. Query caching is separate. I enable shared caching selectively because invalidation, memory, and stale-data behavior must be understood.

### 17. JPQL versus native SQL

**Interview-ready answer:**

JPQL queries the entity model and is portable. Native SQL uses database tables and features directly, useful for vendor-specific or highly tuned queries. For read-heavy endpoints I also use projections so only needed columns are selected.

### 18. Pagination performance

**Interview-ready answer:**

Offset pagination is simple but large offsets become expensive and concurrent inserts can shift results. Keyset pagination uses a stable indexed cursor such as `(created_at, id)` and is better for deep, continuously changing datasets, though it does not support arbitrary page jumps easily.

## Transactions

### 19. What does `@Transactional` do?

**Interview-ready answer:**

It defines a transaction boundary through Spring AOP. The transaction manager begins or joins a transaction before the method and commits or rolls it back afterward. I normally put it on public service methods representing a complete use case.

### 20. Default rollback behavior

**Interview-ready answer:**

By default, Spring rolls back for unchecked exceptions and errors, but not checked exceptions. I configure rollback rules when the business operation requires it rather than wrapping exceptions merely to trigger rollback.

### 21. Transaction propagation

**Interview-ready answer:**

`REQUIRED` joins an existing transaction or creates one and is the default. `REQUIRES_NEW` suspends the current transaction and starts another. `SUPPORTS` joins if present. `MANDATORY` requires one. `NOT_SUPPORTED` runs without one. `NESTED` uses savepoint semantics where supported. I use `REQUIRES_NEW` cautiously because outer rollback will not undo it.

### 22. Isolation levels and anomalies

| Isolation | Prevents | Still may allow |
|---|---|---|
| Read Uncommitted | — | dirty, non-repeatable, phantom reads |
| Read Committed | dirty reads | non-repeatable, phantom reads |
| Repeatable Read | dirty and non-repeatable reads | phantom behavior depends on DB |
| Serializable | standard anomalies | lowest concurrency |

**Interview-ready answer:**

I use the database default unless a business invariant requires stronger isolation. Stronger isolation increases coordination, so I combine an appropriate level with unique constraints, optimistic locking, or explicit locking.

### 23. Why may `@Transactional` not work?

**Interview-ready answer:**

Common causes are self-invocation, a method not invoked through the Spring proxy, unsupported method visibility or final behavior for the proxy setup, the wrong transaction manager, or catching an exception and not rethrowing it. I verify both proxy boundaries and actual database behavior.

### 24. Can a database transaction include a remote HTTP call?

**Interview-ready answer:**

The local database transaction cannot atomically include a normal remote HTTP service. Holding the DB transaction open during the call also increases locks and latency. For cross-service consistency I use patterns such as saga, outbox, idempotency, and compensation.

## SQL

### 25. Primary key, unique key, and foreign key

**Interview-ready answer:**

A primary key uniquely identifies a row and is non-null. A unique constraint enforces uniqueness for another candidate key, with null behavior depending on the database. A foreign key enforces referential integrity between tables.

### 26. What is an index?

**Interview-ready answer:**

An index is a data structure that speeds reads by avoiding full scans, but it consumes space and adds cost to writes. I design composite indexes around actual filters, joins, and sort order, then confirm their use with an execution plan.

### 27. Composite-index order

**Interview-ready answer:**

Order matters. A B-tree composite index is generally most useful from its leftmost columns. I usually put equality predicates first, then range or sort columns, guided by selectivity and the database optimizer rather than a fixed slogan.

### 28. `WHERE` versus `HAVING`

**Interview-ready answer:**

`WHERE` filters rows before grouping, while `HAVING` filters groups after aggregation. I use `WHERE` whenever possible because reducing rows earlier is usually more efficient.

### 29. Inner and outer joins

**Interview-ready answer:**

An inner join returns matching rows. A left join returns all left rows plus matches, with nulls when missing. Right join is the reverse, and full outer join returns unmatched rows from both sides where supported.

### 30. `DELETE`, `TRUNCATE`, and `DROP`

**Interview-ready answer:**

`DELETE` removes selected rows and supports a `WHERE` clause. `TRUNCATE` removes all rows using database-specific DDL-like semantics and is typically faster with different logging and identity behavior. `DROP` removes the object itself. Transaction behavior varies by database, so I do not make universal rollback claims.

### 31. Normalization and denormalization

**Interview-ready answer:**

Normalization reduces duplication and update anomalies by separating data according to dependencies. Denormalization intentionally duplicates or precomputes data for read performance. I keep the source of truth clear and denormalize only with a synchronization strategy.

### 32. How do you troubleshoot a slow query?

**Interview-ready answer:**

I capture the exact SQL and parameters, inspect the execution plan, row estimates, scans, joins, sorts, and locks, then check indexes and data distribution. I reduce selected data or rewrite the query if appropriate, update statistics, test with production-like volume, and measure the end-to-end improvement.

### 33. ACID

**Interview-ready answer:**

Atomicity means all or none of a transaction is applied. Consistency means constraints and invariants are preserved. Isolation controls concurrent transaction interaction. Durability means committed data survives failures according to the database guarantee.

## Common traps

- Treating eager fetching as the N+1 fix
- Calling `save()` after every managed entity update
- Returning entities directly as API DTOs
- Assuming `@Transactional` works on self-invocation
- Holding a DB transaction open across slow remote calls
- Adding indexes without considering write cost or execution plans

