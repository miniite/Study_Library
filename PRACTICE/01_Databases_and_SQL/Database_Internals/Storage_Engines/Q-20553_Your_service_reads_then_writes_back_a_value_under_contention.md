---
type: "practice"
difficulty: "Deep"
---

# Your service reads then writes back a value under contention causing lost updates. Explain how optimistic concurrency with a version column versus SELECT FOR UPDATE (pessimistic) behaves differently at the B-Tree/lock level and which fits high contention.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree Concurrency and Latches](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Optimistic concurrency control using a version column and pessimistic locking via SELECT FOR UPDATE are two strategies to manage concurrent updates in databases, especially when multiple transactions might access and modify the same row. At the B-tree level or lock granularity, these mechanisms behave differently, with significant implications for performance under high contention.

Firstly, let's clarify what each approach entails:

Optimistic Concurrency Control (OCC) via Version Column: This method assumes that conflicts are rare and checks integrity only when a transaction is committed.
Pessimistic Locking (SELECT FOR UPDATE): This technique locks the row immediately to prevent other transactions from modifying it, ensuring consistency.

Now, let's delve into how these mechanisms interact with B-trees and locking at a granular level:

Optimistic Concurrency Control (OCC) with Version Column

In an OCC system using a version column:

Read Phase: A transaction reads the row but doesn't lock it. It also retrieves the current version number.
Write Phase: After modifying the data, the transaction checks if another transaction has modified the same row by comparing the current version with the one stored in the read snapshot. If they differ, a conflict occurs, and the transaction retries or fails.
B-Tree Interaction:
When reading, the database reads from an index (B-tree) but doesn’t acquire any exclusive locks.
If a write operation modifies a value that another transaction has read, it only discovers this conflict during commit time, not immediately.
Pessimistic Locking (SELECT FOR UPDATE)

In pessimistic locking:

Read Phase: A transaction acquires an exclusive lock on the row (or a shared lock if reading), which prevents other transactions from modifying or even reading the row.
Write Phase: The transaction modifies the data and commits, releasing all acquired locks.
B-Tree Interaction:
Immediately upon selecting the row, a shared lock is placed on it at the index level to prevent modifications until commit.
If another transaction attempts to modify the same row, the second transaction will block because of the exclusive lock held by the first.
Comparison Under High Contention

High contention scenarios often involve frequent reads and writes. Let’s break down how each mechanism behaves:

OCC with Version Column

#### Pros
Reduces overhead since rows are not locked.
Allows many readers to proceed without blocking, improving scalability.

#### Cons
Conflicts must be detected during commit, potentially leading to retries and thus reduced performance.
Requires careful implementation of versioning logic.
Pessimistic Locking (SELECT FOR UPDATE)

#### Pros
Ensures strong consistency by preventing any conflicting writes until the transaction commits.

#### Cons
Can lead to significant blocking, especially with many concurrent transactions, degrading performance.
May require more complex transaction management and retry logic.

#### Example Scenario

#### Cons
ider a table orders where each row has an ID, amount, and version column. A user updates the order amount:

Optimistic Concurrency:

Transaction reads the current state: (ID=1, amount=100, version=3).
Modifies to (amount=150) but doesn’t commit.
On commit, it checks SELECT * FROM orders WHERE ID = 1 AND version = 3 for conflict; if any other transaction modified the order in the meantime, it fails.

Pessimistic Locking:

Transaction starts: SELECT FOR UPDATE * FROM orders WHERE ID = 1.
Reads and modifies (amount=150).
Commits, releasing locks on the row.
If another transaction tries to update the same order in between, it will block until the first transaction commits.

#### Closing Summary

Optimistic concurrency control is generally more suitable for high-contention scenarios where frequent reads are common and conflicts can be infrequent. Pessimistic locking ensures strong consistency but can lead to significant blocking and reduced scalability under heavy load.

#### Common Mistakes

Overlooking the need for careful versioning logic in OCC.
Not understanding the implications of lock granularity on performance.
Failing to recognize that pessimistic locking can lead to hotspots if not managed properly.

**Comparison:**

| Transactions | OCC Behavior                    | Pessimistic Locking Behavior         |
|--------------|---------------------------------|-------------------------------------|
| Read Phase   | No lock is held                 | Shared lock on the row              |
| Write Phase  | Check version for conflict      | Exclusive lock on the row            |

**Code:**

```sql
-- Example of OCC with version column
BEGIN;
SELECT * FROM orders WHERE ID = 1 FOR NO KEY UPDATE; -- Read without blocking
UPDATE orders SET amount = 150, version = version + 1 WHERE ID = 1 AND version = 3;
COMMIT;

-- Example of pessimistic locking
BEGIN;
SELECT * FROM orders WHERE ID = 1 FOR UPDATE; -- Lock row until commit
UPDATE orders SET amount = 150 WHERE ID = 1;
COMMIT;
```

**Example input / output:**

```text
-- OCC Example
BEGIN;
SELECT * FROM orders WHERE ID = 1 FOR NO KEY UPDATE; -- Read without blocking
UPDATE orders SET amount = 150, version = version + 1 WHERE ID = 1 AND version = 3;

-- Pessimistic Locking Example
BEGIN;
SELECT * FROM orders WHERE ID = 1 FOR UPDATE; -- Lock row until commit
UPDATE orders SET amount = 150 WHERE ID = 1;
```

**Data:**

```json
{
  "orders": [
    { "ID": 1, "amount": 100, "version": 3 },
    { "ID": 2, "amount": 50, "version": 2 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Read Row
4. Version Match?
5. Commit
6. Retry or Fail
7. Start Transaction
8. Select Row FOR UPDATE
9. Lock Row
10. Write & Commit
11. Start Transaction
12. Select Row FOR NO KEYUPDATE
13. Read without Locking
14. Write & Commit

