---
type: "practice"
difficulty: "Deep"
---

# How do you decide whether to solve a concurrency bug with a higher isolation level, an explicit lock, a unique constraint, or an application-level mutex?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Setting Isolation Levels](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the concurrency issue in a database, deciding between using a higher isolation level, an explicit lock, a unique constraint, or an application-level mutex involves understanding both the nature of the problem and the trade-offs each solution entails. The core idea is to choose the most efficient and maintainable approach that meets the transactional requirements while minimizing performance overhead.

First, it's crucial to understand what the isolation levels (RU/RC/RR) and Serializable actually mean in terms of transaction behavior:

Read Uncommitted (RU): Allows dirty reads, but no locking.
Read Committed (RC): Prevents dirty reads but allows non-repeatable reads and phantom reads.
Repeatable Read (RR): Ensures repeatable read operations within a transaction by defaulting to row-level locks.
Serializable: Enforces strict isolation, preventing any form of concurrency issues but with significant overhead.

Now, let's break down the question step-by-step:

Identify the Concurrency Issue: Determine what exactly is causing the bug—dirty reads, non-repeatable reads, or phantom reads. This will guide your choice among the options.

Assess Performance Impact and Locking Needs:

Higher isolation levels like RC and RR might be necessary if you're dealing with more complex scenarios where simpler solutions are insufficient.
An explicit lock can provide granular control over which rows or tables need to be locked, making it suitable for specific, known contention points.
A unique constraint is a declarative way of ensuring data integrity without the overhead of locking and can be very effective in certain contexts.
An application-level mutex might be needed if the database alone cannot manage the concurrency due to its complexity or limitations.

#### Solution
Selection:

If the issue is with dirty reads, you may need a higher isolation level like RC.
For non-repeatable or phantom reads, RR could solve the problem without excessive locking.
Unique constraints can handle many cases of duplicate data issues efficiently.
Explicit locks are useful when specific rows require protection during certain operations.

#### Cons
ider Application Complexity and Maintainability: Higher isolation levels and explicit locking increase the complexity of your database schema and queries. An application-level mutex, while more complex to implement, might be easier to manage in terms of transaction logic.

#### Closing Summary
The choice between these solutions depends on the nature of the concurrency problem and the performance requirements. Generally, start with simpler declarative methods like unique constraints or explicit locks before opting for higher isolation levels, which can significantly impact system performance due to increased locking overhead.

#### Common Mistakes/Follow-Ups

Not thoroughly understanding the nature of the concurrency issue.
Overlooking application-level solutions in favor of database-only approaches.
Ignoring the trade-offs between simplicity and performance when choosing a solution.

**Comparison:**

```
order	customer_id	status
1	12345	open
```

**Code:**

```sql
-- Example using an explicit lock
BEGIN TRANSACTION;
SELECT * FROM orders WHERE order_id = 1 FOR UPDATE; -- Explicitly lock this row
-- Perform operations that require exclusive access to the locked row
COMMIT;
```

**Example input / output:**

```text
- **Input**:
  - A read operation on the `orders` table, where `order_id = 1`.
- **Output/Response**:
  - The row with `order_id = 1` is locked for this transaction.
```

**Data:**

```json
{
    "order": {
        "id": 1,
        "customer_id": 12345,
        "status": "open"
    }
}
```

**Flow:**

1. No
2. Yes
3. Explicit Lock
4. Higher Isolation
5. Read Operation
6. Isolation Level Needed?
7. Use Unique Constraint
8. Explicit Lock or HigherIsolation?
9. Implement LOCK Command
10. Set Isolation Level toRC/RR/SERIALIZABLE
11. Application-Level Mutex
12. Implement Mutex Logic

