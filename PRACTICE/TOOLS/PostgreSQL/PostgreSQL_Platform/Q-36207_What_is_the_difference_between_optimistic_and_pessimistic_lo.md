---
type: "practice"
difficulty: "Deep"
---

# What is the difference between optimistic and pessimistic locking?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deadlocks and Lock Ordering](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Optimistic and pessimistic locking are two common strategies used to manage concurrent access in databases, particularly in PostgreSQL. In a nutshell, pessimistic locking involves explicitly acquiring locks on data before modifying it, while optimistic locking allows concurrent reads and assumes conflicts will be detected later during updates.

To dive deeper, let's first understand the core concepts of each strategy:

Pessimistic Locking: This approach involves acquiring locks on rows or sets of rows to prevent other transactions from modifying them until the transaction is complete. The main advantage is that it guarantees data consistency and can handle conflicts by blocking access. However, it may lead to deadlocks and waste resources if many transactions are waiting for locks.

Optimistic Locking: This approach assumes that concurrent modifications will be rare and allows multiple transactions to read the same row simultaneously. When an update is attempted, a version number or timestamp is checked against the expected value from earlier reads. If they don't match, it indicates a conflict, and the operation fails.

Let's break down each strategy step-by-step:

Pessimistic Locking
How It Works:
Row-Level Locks: In PostgreSQL, row-level locks can be acquired using SELECT FOR UPDATE or FOR SHARE commands.
Transaction Flow:
A transaction starts and performs a read query.
The database acquires an exclusive lock on the rows being read.
Any subsequent transaction trying to modify these rows will wait until the lock is released.
When the original transaction commits or rolls back, the locks are released.

#### Example
SQL
-- Start a transaction
BEGIN;

-- Read and lock a row for update
SELECT * FROM orders WHERE order_id = 1 FOR UPDATE;

-- Modify the row
UPDATE orders SET status = 'processed' WHERE order_id = 1;
Optimistic Locking
How It Works:
Version Checking: Use version numbers or timestamps to detect conflicts.
Transaction Flow:
A transaction starts and performs a read query, storing the current state of the row (version number).
The transaction makes changes.
When attempting to commit, it checks if the row has been modified since the initial read. If not, the update is allowed; otherwise, an error is raised.

#### Example
SQL
-- Start a transaction
BEGIN;

-- Read and store the version number
SELECT * FROM orders WHERE order_id = 1 FOR NO KEY UPDATE;
-- Store the current version number in a variable

-- Modify the row
UPDATE orders SET status = 'processed', version_number = version_number + 1 WHERE order_id = 1 AND version_number = <stored_version>;

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Common Mistakes

Forgetting to use FOR UPDATE in pessimistic locking can lead to read-only transactions.
Not incrementing the version number correctly in optimistic locking can result in false positives or negatives.

#### Follow-Up
Questions:

How does PostgreSQL handle these locks?
What are some scenarios where one might prefer pessimistic over optimistic locking, and vice versa?

**Code:**

```sql
-- Example of pessimistic locking
 BEGIN;
 SELECT * FROM orders WHERE order_id = 1 FOR UPDATE;
 UPDATE orders SET status = 'processed' WHERE order_id = 1 AND lock_version = 0;

 -- Example of optimistic locking
 BEGIN;
 SELECT * FROM orders WHERE order_id = 1 FOR NO KEY UPDATE INTO my_order;
 UPDATE orders SET status = 'processed', version_number = version_number + 1 WHERE order_id = 1 AND version_number = 0;
```

