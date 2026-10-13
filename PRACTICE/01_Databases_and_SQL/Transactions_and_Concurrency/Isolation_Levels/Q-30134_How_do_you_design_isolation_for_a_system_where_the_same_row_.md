---
type: "practice"
difficulty: "Deep"
---

# How do you design isolation for a system where the same row is a hot contended counter (likes/views) at web scale? Why is any pessimistic level the wrong tool here?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Isolation for Shared Subsystems](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_for_Shared_Subsystems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When designing isolation levels for a system handling a highly contended counter like likes or views, we need to balance between maintaining data consistency and avoiding unnecessary performance bottlenecks. In a web-scale application, pessimistic locking (RU/RC) can lead to significant contention issues. This answer explores why optimistic locking (RR/Serializable) is more appropriate in such scenarios.

#### Triage

The question probes understanding of isolation levels and their implications on system performance at scale. Specifically, it challenges the assumption that pessimistic locking is always the right tool. The key concepts are:

Isolation Levels: Understanding RU, RC, RR, and Serializable.
Contention Handling: How different isolation levels handle concurrent access to a shared resource (like a counter).
System Performance: Trade-offs between consistency and performance.

#### First-Principles Solution
1. Understanding Isolation Levels
Read Uncommitted (RU): Allows dirty reads, where uncommitted transactions can be read.
Read Committed (RC): Prevents reading dirty data but allows phantom reads (new rows inserted during the transaction).
Repeatable Read (RR): Ensures that a transaction sees consistent data; prevents both dirty and non-repeatable reads. However, it locks rows to prevent other transactions from modifying them.
Serializable: The highest isolation level, providing full isolation by locking all accessed data until the transaction completes.
2. Contention Analysis

In a highly contended counter scenario, every read/write operation becomes a bottleneck because RR and Serializable both lock rows, preventing concurrent access. This leads to:

Lock Contention: High number of locks, which can degrade performance as more transactions are waiting.
Deadlocks: Potential for transactions getting stuck in a deadlock loop if they acquire locks in different orders.
3. Optimistic Locking

Optimistic locking (RR) is better suited here because it:

Avoids Unnecessary Locks: Only locks rows when a write operation occurs, reducing contention.
Minimizes Blocking: Transitions quickly from read to write operations without blocking other transactions.
Detailed Solution
Step 1: Define the Problem

We are designing a system that needs to handle concurrent access to a counter (e.g., likes/views). Each increment or decrement of this counter should be atomic and consistent, but we must minimize locking overhead.

Step 2: Choose an Appropriate Isolation Level

Given the high contention:

Serializable is too restrictive, as it locks all accessed data, leading to significant blocking.
Read Committed (RC) is better than RR because it prevents dirty reads and allows concurrent reads without locks. However, it might still suffer from non-repeatable reads if new rows are inserted.
Step 3: Implement Optimistic Locking

We should use a combination of:

Optimistic Locking: Allow multiple transactions to read the counter simultaneously.
Timestamp-Based Validation: Each transaction increments or decrements the counter, then checks the timestamp. If the timestamp has changed (indicating another transaction modified the value), retry the operation.
Step 4: Example Implementation
SQL
-- Assume we have a table `counter` with columns `id`, `value`, and `timestamp`
BEGIN TRANSACTION;

SELECT * FROM counter WHERE id = ? FOR UPDATE OF value;
// Check if timestamp has changed; if yes, retry the transaction
UPDATE counter SET value = value + 1 WHERE id = ? AND timestamp = ?;

COMMIT;

#### Closing Summary

For a highly contended counter at web scale, optimistic locking (RR) is preferred over pessimistic levels because it minimizes contention and blocking. However, implementing this requires careful validation to ensure data integrity.

#### Common Mistakes/Follow-Ups
Overlooking Timestamps: Ignoring timestamp checks can lead to race conditions.
Underestimating Concurrency: Failing to account for high concurrency scenarios where retries become a bottleneck.
Incorrect Locking: Using pessimistic locking (RU/RC) unnecessarily, leading to performance issues.

**Code:**

```sql
BEGIN TRANSACTION;

SELECT * FROM counter WHERE id = ? FOR UPDATE OF value;
-- Check if timestamp has changed; if yes, retry the transaction
UPDATE counter SET value = value + 1 WHERE id = ? AND timestamp = ?;

COMMIT;
```

**Example input / output:**

```text
-- Example Input
BEGIN TRANSACTION;

SELECT * FROM counter WHERE id = 1 FOR UPDATE OF value;

-- Check if timestamp has changed; if yes, retry the transaction
UPDATE counter SET value = value + 1 WHERE id = 1 AND timestamp = '2023-10-01T12:00:00';

COMMIT;
```

