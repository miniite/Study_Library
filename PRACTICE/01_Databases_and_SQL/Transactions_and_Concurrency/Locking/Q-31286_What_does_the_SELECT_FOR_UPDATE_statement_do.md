---
type: "practice"
difficulty: "Intermediate"
---

# What does the SELECT ... FOR UPDATE statement do?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Row Locks Two-phase Locking and Lock Footprint](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Row_Locks_Two-phase_Locking_and_Lock_Footprint.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The SELECT ... FOR UPDATE statement in SQL is used to lock specific rows in a table, ensuring that only one transaction can modify those rows at a time. This mechanism prevents other transactions from modifying or deleting these locked rows until the current transaction commits or rolls back.

#### Triage

What the Question is Asking: The question seeks an understanding of how SELECT ... FOR UPDATE works and its purpose in database operations.
What the Interviewer is Probing: The interviewer wants to assess your grasp of concurrency control and locking mechanisms, which are crucial for maintaining data integrity in multi-user environments.

Step-by-Step Solution From First Principles:

Understanding the Statement:

SELECT ... FOR UPDATE locks the selected rows immediately after they have been fetched by the query.
These locks remain until the transaction that performed the select either commits or rolls back.

Locking Behavior:

The exact type of lock (e.g., shared, exclusive) depends on the isolation level and database system in use.
In many systems, it will create an "update" or "shared update" lock, which prevents other transactions from modifying those rows but allows other transactions to read them.

Usage Scenarios:

Preventing Dirty Reads: By locking rows before performing a modification, you ensure that no other transaction can modify the data in between the SELECT and UPDATE.
Ensuring Data Integrity: This is particularly useful when working with financial transactions or any application where data integrity is critical.

Impact on Performance:

Locking can slow down performance, especially if many rows are locked frequently.
Proper indexing can help minimize the impact of locking by reducing the number of rows that need to be scanned and locked.

Common Pitfalls:

Not considering the isolation level settings when using FOR UPDATE.
Failing to release locks promptly after transactions complete, leading to potential deadlocks or excessive resource consumption.
Incorrectly designing queries that lock large numbers of rows unnecessarily.

**Code:**

```sql
-- Example usage in SQL
SELECT * FROM orders WHERE order_id = 123 FOR UPDATE;
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Execute SELECT ... FOR UPDATE
5. Rows Selected?
6. Lock Rows
7. Commit/Abort Transaction
8. End Transaction WithoutLocking

