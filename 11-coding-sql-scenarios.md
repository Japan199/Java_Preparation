# Coding, SQL, Debugging, and Scenario Practice

Explain the approach, complexity, edge cases, and tests before or while coding. Prefer a clear correct solution over a clever one.

## Java coding questions

### 1. First non-repeated character

```java
static Optional<Character> firstNonRepeated(String input) {
    Map<Character, Integer> counts = new LinkedHashMap<>();
    input.chars()
        .mapToObj(c -> (char) c)
        .forEach(c -> counts.merge(c, 1, Integer::sum));
    return counts.entrySet().stream()
        .filter(e -> e.getValue() == 1)
        .map(Map.Entry::getKey)
        .findFirst();
}
```

**Complexity:** O(n) time and O(k) space.

### 2. Check whether two strings are anagrams

```java
static boolean areAnagrams(String left, String right) {
    if (left == null || right == null) return false;
    char[] a = left.replaceAll("\\s+", "").toLowerCase().toCharArray();
    char[] b = right.replaceAll("\\s+", "").toLowerCase().toCharArray();
    Arrays.sort(a);
    Arrays.sort(b);
    return Arrays.equals(a, b);
}
```

**Discussion:** Clarify case, whitespace, punctuation, and Unicode rules. Sorting is O(n log n); a frequency map can be O(n).

### 3. Find duplicates and frequency

```java
Map<Integer, Long> frequency = numbers.stream()
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ));

List<Integer> duplicates = frequency.entrySet().stream()
    .filter(e -> e.getValue() > 1)
    .map(Map.Entry::getKey)
    .toList();
```

### 4. Second-highest distinct number

```java
static OptionalInt secondHighest(int[] values) {
    return Arrays.stream(values)
        .distinct()
        .boxed()
        .sorted(Comparator.reverseOrder())
        .skip(1)
        .mapToInt(Integer::intValue)
        .findFirst();
}
```

**Follow-up:** A one-pass solution uses two variables and O(1) extra space.

### 5. Reverse words, not characters

```java
static String reverseWords(String input) {
    List<String> words = Arrays.asList(input.trim().split("\\s+"));
    Collections.reverse(words);
    return String.join(" ", words);
}
```

### 6. Palindrome check

```java
static boolean isPalindrome(String input) {
    int left = 0, right = input.length() - 1;
    while (left < right) {
        if (input.charAt(left++) != input.charAt(right--)) return false;
    }
    return true;
}
```

**Complexity:** O(n) time, O(1) additional space.

### 7. Longest substring without repeating characters

```java
static int longestUniqueSubstring(String text) {
    Map<Character, Integer> lastSeen = new HashMap<>();
    int start = 0, best = 0;
    for (int end = 0; end < text.length(); end++) {
        char c = text.charAt(end);
        if (lastSeen.containsKey(c)) {
            start = Math.max(start, lastSeen.get(c) + 1);
        }
        lastSeen.put(c, end);
        best = Math.max(best, end - start + 1);
    }
    return best;
}
```

**Complexity:** O(n) time, O(k) space.

### 8. Merge overlapping intervals

```java
static List<int[]> merge(int[][] intervals) {
    if (intervals.length == 0) return List.of();
    Arrays.sort(intervals, Comparator.comparingInt(a -> a[0]));
    List<int[]> result = new ArrayList<>();
    int[] current = intervals[0].clone();
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i][0] <= current[1]) {
            current[1] = Math.max(current[1], intervals[i][1]);
        } else {
            result.add(current);
            current = intervals[i].clone();
        }
    }
    result.add(current);
    return result;
}
```

**Complexity:** O(n log n), dominated by sorting.

### 9. Producer-consumer design

**Interview-ready answer:**

I use a bounded `BlockingQueue`. Producers call `put()` and consumers call `take()`, so coordination and backpressure are explicit. I define shutdown using interruption, a poison message, or executor lifecycle. I would not manually implement wait/notify unless specifically asked.

### 10. Thread-safe singleton

```java
public final class AppConfig {
    private AppConfig() {}

    private static class Holder {
        private static final AppConfig INSTANCE = new AppConfig();
    }

    public static AppConfig instance() {
        return Holder.INSTANCE;
    }
}
```

**Interview note:** In Spring applications, prefer a container-managed singleton bean.

### 11. LRU cache approach

**Interview-ready answer:**

An LRU cache needs O(1) lookup and O(1) recency updates, so I combine a hash map with a doubly linked list. The map locates nodes; the list keeps most-recent to least-recent order. `LinkedHashMap` with access order is the concise Java implementation for a local non-concurrent version.

### 12. Employee stream exercises

```java
// Average salary by department
Map<String, Double> averageSalary = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

// Names sorted by salary descending
List<String> names = employees.stream()
    .sorted(Comparator.comparingDouble(Employee::getSalary).reversed())
    .map(Employee::getName)
    .toList();
```

## SQL questions

Assume `employee(id, name, department_id, salary, manager_id)` and `department(id, name)`.

### 13. Second-highest distinct salary

```sql
SELECT MAX(salary) AS second_highest
FROM employee
WHERE salary < (SELECT MAX(salary) FROM employee);
```

Using a window function:

```sql
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank_no
    FROM employee
) ranked
WHERE rank_no = 2;
```

### 14. Nth-highest salary

```sql
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rank_no
    FROM employee
) ranked
WHERE rank_no = :n;
```

### 15. Highest-paid employee in each department

```sql
SELECT department_id, id, name, salary
FROM (
    SELECT e.*,
           DENSE_RANK() OVER (
               PARTITION BY department_id
               ORDER BY salary DESC
           ) AS rank_no
    FROM employee e
) ranked
WHERE rank_no = 1;
```

### 16. Employees earning more than their manager

```sql
SELECT e.name AS employee_name, m.name AS manager_name
FROM employee e
JOIN employee m ON m.id = e.manager_id
WHERE e.salary > m.salary;
```

### 17. Find duplicate names

```sql
SELECT name, COUNT(*) AS duplicate_count
FROM employee
GROUP BY name
HAVING COUNT(*) > 1;
```

### 18. Departments with no employees

```sql
SELECT d.id, d.name
FROM department d
LEFT JOIN employee e ON e.department_id = d.id
WHERE e.id IS NULL;
```

### 19. Delete duplicates while keeping the lowest ID

```sql
WITH ranked AS (
    SELECT id,
           ROW_NUMBER() OVER (
               PARTITION BY name, department_id
               ORDER BY id
           ) AS row_no
    FROM employee
)
DELETE FROM employee
WHERE id IN (SELECT id FROM ranked WHERE row_no > 1);
```

**Interview note:** Syntax differs by database. Preview rows and use a transaction before a destructive statement.

### 20. `ROW_NUMBER`, `RANK`, and `DENSE_RANK`

**Interview-ready answer:**

`ROW_NUMBER` gives every row a unique sequence. `RANK` gives ties the same rank and leaves gaps. `DENSE_RANK` gives ties the same rank without gaps. For the second distinct salary, `DENSE_RANK` normally expresses the requirement.

## Scenario questions

### 21. API became slow after a release. What do you do?

**Interview-ready answer:**

I compare latency, error, traffic, saturation, and dependency metrics before and after the release and inspect traces for the slow span. I check query plans, connection pools, thread pools, GC, and external calls. If user impact is high, I roll back or disable the change, then reproduce, fix, load-test, and add a regression test and alert.

### 22. Database connection pool is exhausted.

**Interview-ready answer:**

I check active versus idle connections, acquisition time, slow queries, transaction duration, leaked connections, and database capacity. I stop overload or scale only if the database can support it. The root fix may be query tuning, shorter transaction scope, correct timeouts, leak removal, or appropriately sized pools—not simply increasing the pool.

### 23. Messages are processed twice.

**Interview-ready answer:**

Duplicate delivery is normal in at-least-once systems. I confirm offset and failure behavior, then make the business effect idempotent using an event ID and unique constraint or an idempotent state transition. I commit after successful processing and test crashes between effect and acknowledgement.

### 24. One downstream service is failing.

**Interview-ready answer:**

I enforce a short timeout, fail fast with a circuit breaker when failure is sustained, isolate capacity with a bulkhead, and retry only safe transient failures with backoff and jitter. I return a truthful fallback for optional data or a clear error for critical operations, while metrics and traces identify the dependency.

### 25. Concurrent requests oversell inventory.

**Interview-ready answer:**

I enforce the invariant atomically in the database, for example an update conditioned on sufficient quantity and verify the affected-row count, or use optimistic locking with retry. A read-then-write check without coordination is unsafe. For distributed reservations I use an expiring reservation model and reconciliation.

### 26. An API receives huge traffic spikes.

**Interview-ready answer:**

I protect the system with gateway rate limits, bounded queues and concurrency, autoscaling within downstream capacity, caching for safe reads, and load shedding for noncritical work. I measure the bottleneck first and use performance tests; unlimited buffering or retries make spikes worse.

### 27. Data differs between two services.

**Interview-ready answer:**

I identify the source of truth and inspect event publication, consumer lag, failures, ordering, and schema compatibility. I replay or reconcile safely using idempotent operations, fix the broken propagation path, and add monitoring for lag and business-level inconsistency.

### 28. A production-only memory leak appears.

**Interview-ready answer:**

I monitor heap after full GC, capture heap dumps at safe points, compare retained objects and paths to GC roots, and inspect caches, static collections, listeners, and thread locals. I mitigate memory pressure, fix the retaining lifecycle or bound, and verify with a production-like soak test.

## Coding-round checklist

- Clarify nulls, empty input, duplicates, case, order, and size.
- State brute force, then improve only where valuable.
- Select the data structure deliberately.
- State time and space complexity.
- Use meaningful names and small methods.
- Test normal, boundary, duplicate, and invalid cases.
- Do not force streams when a loop is clearer.

