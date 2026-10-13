---
type: "practice"
difficulty: "Deep"
---

# How do you approach a deadlock where the two statements are actually a SELECT and an UPDATE on the same table via different indexes?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Deadlock Causes and Diagnosis](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Deadlocks/Deadlock_Causes_and_Diagnosis.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address deadlocks involving SELECT and UPDATE operations on the same table through different indexes, we need to understand how SQL handles transactions and locking, as well as the specific conditions that can lead to a deadlock. Here's a step-by-step approach:

Understand the Core Issue: Deadlocks occur when two or more transactions are waiting for each other to release locks on resources they both require. In this case, we have a SELECT statement and an UPDATE statement on the same table but via different indexes.

Identify the Locking Behavior:

A SELECT with a lock (e.g., SELECT ... FOR UPDATE) places a shared lock (S-lock), which allows other transactions to read the data but prevents them from acquiring an exclusive lock (X-lock).
An UPDATE statement acquires an exclusive lock on the row(s) it needs to modify.

Analyze the Scenario:

Suppose Transaction A has a SELECT ... FOR UPDATE on Index1 and starts processing.
Meanwhile, Transaction B performs an UPDATE operation using Index2, which requires a lock that overlaps with what A already holds (e.g., both are interested in rows where id = 100).

Resolve the Deadlock:

The database system will detect the deadlock and automatically resolve it by rolling back one of the transactions to break the cycle.
By default, most databases use a heuristic algorithm like the One-Serializable Version (OSV) or the First-Come-First-Served (FCFS) approach to decide which transaction should be rolled back. This is usually done on a per-database or per-session basis.

Preventive Measures:

Optimize Queries: Ensure that queries are optimized and indexes are used efficiently.
Reduce Transaction Durations: Minimize the duration of transactions by reducing the number of operations and ensuring they run as quickly as possible.
Use Isolation Levels: Adjust isolation levels to balance between concurrency and consistency. For example, using READ COMMITTED or REPEATABLE READ instead of SERIALIZABLE.

Post-Deadlock Handling:

After a deadlock is detected, the system will log which transaction was rolled back.
Application logic should handle this gracefully by retrying the transaction if necessary.

**Code:**

```sql
-- Example SELECT FOR UPDATE statement
SELECT * FROM orders WHERE customer_id = 100 FOR UPDATE;

-- Example UPDATE statement
UPDATE orders SET status = 'processed' WHERE order_id = 200;
```

**Example input / output:**

```text
Transaction A:
- Selects rows with customer_id = 100 using Index1 and places S-lock.

Transaction B:
- Updates rows where order_id = 200 using Index2, which requires an X-lock on the same row as Transaction A's S-lock.

Deadlock Detection:
- Database detects deadlock between Transaction A and B.
- One transaction is rolled back to break the cycle.

Outcome:
- Transaction A or B is rolled back with an error message indicating a deadlock was detected.
```

**Flow:**

1. SELECT ... FOR UPDATE
2. UPDATE
3. Deadlock detected
4. Transaction A
5. S-lock on customer_id = 100
6. Update order_id = 200, X-lockconflicts with S-lock
7. Rollback Transaction A/B

