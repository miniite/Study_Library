---
type: "practice"
difficulty: "Deep"
---

# How do isolation levels interact with SELECT ... FOR SHARE versus FOR UPDATE? When would FOR SHARE still let a write skew slip through?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Setting Isolation Levels](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Isolation levels in databases determine how transactions see each other's data, and FOR SHARE and FOR UPDATE are specific locking mechanisms that interact with isolation levels to control concurrency. While FOR SHARE locks rows for reading but allows others to read or lock them too, FOR UPDATE locks rows both for reading and writing, preventing concurrent writes from modifying the same row. In terms of preventing write skew, SERIALIZABLE is crucial because it ensures that transactions behave as if they were executed serially.

#### Triage

The question delves into how isolation levels (RU, RC, RR, and Serializable) interact with different locking mechanisms like FOR SHARE and FOR UPDATE. Specifically, it asks when a write skew might still occur despite using FOR SHARE.

What the Question is Asking:
Isolation Levels: Different levels determine how transactions see each other's data.
Locking Mechanisms (FOR SHARE, FOR UPDATE): These control concurrency by locking rows for different purposes.

#### Assumptions
and Clarifications:
We are considering scenarios in a multi-user environment where multiple transactions might be running simultaneously.
The focus is on understanding when FOR SHARE might not prevent write skew, despite the isolation level used.

#### First-Principles Solution
Isolation Levels Explained:
Read Uncommitted (RU): Dirty reads are allowed; changes made by other transactions can be read before they are committed.
Read Committed (RC): Prevents dirty reads but allows non-repeatable reads and phantom reads.
Repeatable Read (RR): Ensures repeatable reads, preventing non-repeatable reads but allowing phantom reads.
Serializable: Ensures serializability of transactions by preventing all types of concurrency issues.
Locking Mechanisms:
FOR SHARE locks rows for reading only; other transactions can still read or lock the same rows.
FOR UPDATE locks rows both for reading and writing, blocking any changes until the transaction completes.
Interaction Between Isolation Levels and Locking:
Read Uncommitted (RU): FOR SHARE is meaningless as no locking is performed.
Read Committed (RC): FOR SHARE can still allow write skew if concurrent writes occur without proper locking.
Repeatable Read (RR): FOR SHARE helps prevent dirty reads but may not fully prevent write skew.
Serializable: Ensures all transactions behave as if they were executed serially, preventing any form of write skew.
Write Skew Scenario:

Write skew occurs when a transaction's operation depends on the state of multiple rows, and concurrent writes can alter that state in a way that invalidates the transaction's logic. FOR SHARE might not prevent this because it only locks for reading, allowing other transactions to modify those rows.

#### Example

#### Cons
ider a database with an order table where you want to ensure that a customer cannot place two orders of the same item at the same time:

SQL
-- Transaction 1 (T1)
BEGIN;
SELECT * FROM orders WHERE item_id = 1 FOR SHARE;  -- Locks the row for reading
INSERT INTO orders (customer_id, item_id) VALUES (123, 1);  -- This might fail if T2 commits first

-- Concurrent Transaction (T2)
BEGIN;
UPDATE orders SET quantity = quantity - 1 WHERE item_id = 1 FOR UPDATE;  -- Locks the row for writing
COMMIT;

-- Back to T1
INSERT INTO orders (customer_id, item_id) VALUES (123, 1);  -- May fail due to concurrent update in T2

In this scenario, T2 updates the quantity before T1 inserts a new order, causing write skew. Even with FOR SHARE, T1 might still see an outdated state of the row when inserting a new order.

#### Closing Summary

FOR SHARE and FOR UPDATE are powerful tools for controlling concurrency but must be used in conjunction with appropriate isolation levels to prevent issues like write skew. Serializable is crucial as it ensures that all transactions behave as if they were executed serially, thereby preventing any form of write skew.

#### Common Mistakes
Underestimating the Need for Locks: Assuming FOR SHARE will always be sufficient.
Overlooking Serializability: Failing to recognize the importance of ensuring that transactions do not interfere with each other's logic.

#### Follow-Ups
How does sharding affect this issue?
What are some strategies to handle write skew in distributed systems?

**Comparison:**

| Item ID | Quantity |
|---------|----------|
| 1       | 4        |

**Code:**

```sql
BEGIN;
SELECT * FROM orders WHERE item_id = 1 FOR SHARE;  -- Locks the row for reading
INSERT INTO orders (customer_id, item_id) VALUES (123, 1);  -- This might fail if T2 commits first

-- Concurrent Transaction (T2)
BEGIN;
UPDATE orders SET quantity = quantity - 1 WHERE item_id = 1 FOR UPDATE;  -- Locks the row for writing
COMMIT;

-- Back to T1
INSERT INTO orders (customer_id, item_id) VALUES (123, 1);  -- May fail due to concurrent update in T2
```

**Example input / output:**

```text
-- Transaction T1
BEGIN;
SELECT * FROM orders WHERE item_id = 1 FOR SHARE;  -- Locks the row for reading

-- Concurrent Transaction T2
BEGIN;
UPDATE orders SET quantity = quantity - 1 WHERE item_id = 1 FOR UPDATE;  -- Locks the row for writing
COMMIT;

-- Back to T1
INSERT INTO orders (customer_id, item_id) VALUES (123, 1);  -- May fail due to concurrent update in T2

-- Output:
Transaction T1 may encounter a failure when trying to insert a new order due to the concurrent update by T2.
```

**Data:**

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INT,
    item_id INT,
    quantity INT
);
INSERT INTO orders (customer_id, item_id, quantity) VALUES (123, 1, 5);
```

**Flow:**

1. May Fail
2. Transaction T1 Starts
3. SELECT * FROM orders WHEREitem_id = 1 FOR SHARE
4. Insertion Attempt
5. Write Skew Occurs
6. Transaction T2 Starts
7. UPDATE orders SET quantity =quantity - 1 WHERE item_id =1 FOR UPDATE
8. Commits Transaction
9. Transaction T1 Resumes
10. Insertion Attempt Succeeds

