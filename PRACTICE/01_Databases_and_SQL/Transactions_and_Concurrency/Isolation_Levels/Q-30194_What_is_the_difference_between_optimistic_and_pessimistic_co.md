---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between optimistic and pessimistic concurrency control, and how do isolation levels map onto each approach?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Isolation with Read Replicas](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Setting_Isolation_Levels/Isolation_with_Read_Replicas.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Optimistic and pessimistic concurrency control are two fundamental approaches in managing concurrent transactions to ensure data consistency without overly compromising performance or scalability. Let’s break down both methods, explore how they relate to isolation levels (RU/RC/RR/Serializable), and discuss their trade-offs.

TL;DR
Pessimistic Concurrency Control (PCC): Locks are acquired before a transaction starts, preventing other transactions from modifying the same data until the current transaction completes.
Optimistic Concurrency Control (OCC): No locks are held initially. After a transaction has modified some data and attempts to commit, it checks for conflicts with concurrent updates made by others.

#### Triage
Question Breakdown

The question is asking you to explain two key concurrency control methods and how they align with database isolation levels. It seeks an understanding of both the theoretical concepts and practical implications of these approaches.

#### First-Principles Solution
Pessimistic Concurrency Control (PCC)
Initial State: Before a transaction starts, all data that might be accessed is locked.
Execution: The transaction runs without worrying about conflicts since it has exclusive access to the data.
Commit: When the transaction commits, the locks are released.

#### Pros

Simple and effective in preventing conflicts.
Ensures no dirty reads or lost updates.

#### Cons

Can lead to high lock contention and performance bottlenecks.
May not scale well with many concurrent transactions.
Optimistic Concurrency Control (OCC)
Initial State: No locks are held until a transaction attempts to commit its changes.
Execution: The transaction runs freely, allowing other transactions to access the data without being blocked.
Commit Check: After committing, the system checks if any other transaction has modified the same data in a conflicting manner.

#### Pros

Better for scalable systems where conflicts are rare or predictable.
Reduces lock overhead and improves performance by avoiding unnecessary locks.

#### Cons

More complex to implement due to additional conflict resolution logic.
Risk of dirty reads, lost updates, and phantom reads if not handled properly.
Mapping Isolation Levels

Isolation levels in databases provide a framework for managing the concurrency control methods described above. Each isolation level represents a different balance between transaction isolation and performance:

Read Uncommitted (RU):

Lowest isolation.
No locks are held or checked, leading to potential dirty reads.

Read Committed (RC):

Basic level of isolation.
Prevents dirty reads but allows phantom reads and non-repeatable reads.

Repeatable Read (RR):

Ensures that once a transaction has read a piece of data, it will always see the same value until the transaction ends.
Pessimistic in nature, often implemented with row-level locking.

Serializable:

Highest isolation level.
Guarantees that concurrent transactions do not interfere with each other’s execution order.
Often achieved using pessimistic concurrency control techniques like multi-version concurrency control (MVCC).

#### Common Mistakes
and Follow-Ups
Mistake: Confusing the exact definitions of PCC and OCC. Make sure to clearly distinguish between them based on their initial state and commit behavior.

#### Follow-Up
Discuss real-world scenarios where one might prefer PCC over OCC, or vice versa. For example, read-intensive applications with few write operations may benefit from OCC due to lower lock overhead.

#### Closing Summary

In summary, pessimistic concurrency control uses locks to prevent conflicts upfront, while optimistic concurrency control allows transactions to run freely and then resolves any conflicts during commit. Each has its place depending on the application’s requirements for performance and data consistency. Isolation levels provide a way to fine-tune these approaches based on specific use cases.

#### Common Mistakes
Failing to differentiate: Between PCC (locks before commit) and OCC (no locks until commit).
Overlooking implications: Of isolation level settings in practice, such as the trade-offs between performance and data integrity.

**Comparison:**

```
Isolation Level	Features	Pros	Cons
Read Uncommitted	Lowest isolation	Fastest performance	Dirty reads possible
Read Committed	Prevents dirty reads	Simple to implement	Allows phantom reads and non-repeatable reads
Repeatable Read	Ensures consistent results across transactions	Better data integrity than RC	Higher lock contention due to row-level locking
Serializable	Highest isolation, no conflicts allowed	Strongest consistency guarantees	Most overhead in terms of performance
```

**Code:**

```sql
-- Example of pessimistic locking using SELECT FOR UPDATE
BEGIN;
SELECT * FROM orders WHERE order_id = 1 FOR UPDATE; -- Locks the row until transaction commit

-- Example of optimistic locking with version number in schema
CREATE TABLE orders (
    id INT PRIMARY KEY,
    item_name VARCHAR(255),
    quantity INT NOT NULL DEFAULT 0 CHECK (quantity >= 0),
    version INT NOT NULL DEFAULT 1
);
```

**Example input / output:**

```text
Input:
- User A tries to update order_id = 1 (laptop) with quantity = 98

Output:
- User A's transaction checks version and finds it matches the current version in the database.
- Transaction commits successfully, updating both the row and version number.

Input:
- Concurrently, User B also tries to update order_id = 1 (laptop) with quantity = 97

Output:
- User B's transaction finds that its version does not match the current version in the database due to User A's change.
- Therefore, User B's transaction fails and rolls back.
```

**Data:**

```json
{
  "orders": [
    { "id": 1, "item_name": "Laptop", "quantity": 100, "version": 1 },
    { "id": 2, "item_name": "Mouse", "quantity": 500, "version": 3 }
  ]
}
```

**Flow:**

1. Start Transaction
2. Isolation Level?
3. Read Uncommitted - No locks,dirty reads possible
4. Read Committed - Locks afterread, prevents dirty reads
5. Repeatable Read - Locks untilcommit, no phantom reads ornon-repeatable reads
6. Serializable - Ensures order ofoperations, highest overhead
7. Optimistic ConcurrencyControl: No locks initially,check at commit time.
8. Pessimistic ConcurrencyControl: Lock data beforecommitting.
9. Conflict Check?
10. Commit Success - Update andrelease lock.
11. Commit Failure - Rollbacktransaction due to conflict.
12. Conflict Check?
13. Commit Success - Use MVCCor locking to ensureconsistency.
14. Commit Failure - Resolveconflicts using rollback or retrymechanism.

