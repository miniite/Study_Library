---
type: "practice"
difficulty: "Deep"
---

# What is the lost update problem in concurrent transactions?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Lock Holding and Optimistic Concurrency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The lost update problem occurs when two or more transactions access and modify the same data concurrently, without proper concurrency control mechanisms, leading to inconsistent database states. This issue can happen even if each transaction individually appears correct but collectively results in a failure of the overall application's integrity.

#### Triaging the Question

This question delves into understanding how concurrent operations on shared data can lead to problematic outcomes despite individual transactions being valid. The core idea here is that concurrent access without proper isolation levels or locking mechanisms can result in lost updates, which can corrupt your database state and violate application requirements.

#### First-Principles Solution

Understanding the Problem: Consider a scenario where Transaction T1 reads data D at time t0, modifies it to M1, and attempts to update D later but finds that another transaction, T2, has already modified D to M2. If T1 updates D as M1, the original change M2 made by T2 will be lost. This is the essence of the lost update problem.

Isolation Levels: Different levels of isolation (Read Uncommitted, Read Committed, Repeatable Read, Serializable) provide varying degrees of protection against such issues.

Read Uncommitted: No locking; all updates are visible to all transactions.
Read Committed: Only committed changes are visible, but the transaction might read uncommitted data between the start and end of its execution.
Repeatable Read: Ensures that once a value is read by a transaction, any subsequent reads within the same transaction will see the same values, even if other transactions commit new versions of those values.
Serializable: The highest level of isolation, ensuring all transactions are serialized such that their effects are visible to no more than one transaction at a time.

Locking Mechanisms: To prevent lost updates, you can use different types of locks:

Row-Level Locks: Lock individual rows.
Table-Level Locks: Lock the entire table.
Pessimistic vs Optimistic Locking: Pessimistic locking involves explicitly acquiring a lock before modifying data; optimistic locking assumes no conflict and only acquires a lock at commit time.

#### Example Scenario

SQL
-- Transaction T1
BEGIN TRANSACTION;
SELECT item_id, quantity FROM inventory WHERE item_id = 1 FOR UPDATE;
UPDATE inventory SET quantity = quantity - 5 WHERE item_id = 1;
COMMIT;

-- Simultaneous transaction T2
BEGIN TRANSACTION;
SELECT item_id, quantity FROM inventory WHERE item_id = 1 FOR UPDATE;
UPDATE inventory SET quantity = quantity - 3 WHERE item_id = 1;
COMMIT;

In this case, if both transactions run concurrently and commit their updates, one of the updates might overwrite the other, leading to a lost update.

#### Closing Summary

The lost update problem arises from concurrent transactions modifying shared data without proper isolation or locking mechanisms. To mitigate this issue, database systems implement various isolation levels and locking strategies to ensure that transactions do not interfere with each other in ways that violate application requirements.

#### Common Mistakes/Follow-Ups
Misunderstanding Isolation Levels: Failing to understand the implications of different isolation levels can lead to incorrect assumptions about transaction behavior.
Over-Locking: Excessive use of locks can degrade performance, leading to deadlocks and other issues.
Not Considering Deadlock Avoidance: Ensuring that transactions acquire locks in a consistent order can prevent deadlock but also requires careful planning.

Would you like me to elaborate on any specific part or explore another related concept?

**Comparison:**

| Item ID | Quantity |
|---------|----------|
| 1       | 48       |

**Code:**

```sql
BEGIN TRANSACTION;
SELECT item_id, quantity FROM inventory WHERE item_id = 1 FOR UPDATE;
UPDATE inventory SET quantity = quantity - 5 WHERE item_id = 1;
COMMIT;

BEGIN TRANSACTION;
SELECT item_id, quantity FROM inventory WHERE item_id = 1 FOR UPDATE;
UPDATE inventory SET quantity = quantity - 3 WHERE item_id = 1;
COMMIT;
```

**Example input / output:**

```text
Input:
- Transaction T1 starts and reads `D` at time `t0`.
- Simultaneously, Transaction T2 starts and also reads `D` but finds the updated value after `T1`'s read.
- Both transactions modify `D` and try to update it.

Output:
- If both commit without proper locking, one transaction's change is lost. For instance, if `M2` by `T2` overwrites `M1` by `T1`, the final value of `D` might be incorrect.
```

**Data:**

```json
{
  "inventory": [
    {"item_id": 1, "quantity": 50},
    {"item_id": 2, "quantity": 40}
  ]
}
```

**Flow:**

1. Transaction T1 starts
2. Read Lock on D
3. Read D at t0
4. Modify D to M1
5. Write Lock on D
6. Commit update of M1
7. Transaction T2 starts
8. Read Lock on D
9. Read D at t0'
10. Modify D to M2
11. Write Lock on D
12. Conflict detected, rollback orhandle conflict

