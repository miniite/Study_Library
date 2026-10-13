---
type: "practice"
difficulty: "Basic"
---

# Why is durability often described as surviving a power failure or crash?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ACID Durability and Consistency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/ACID_Durability_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Durability in database transactions ensures that once a transaction is committed, it remains permanently recorded and won't be lost even if there's a sudden interruption like a power failure or system crash. This concept is fundamental to ensuring data integrity and the reliability of our databases.

In this context, durability means that after a transaction completes successfully (i.e., it has been marked as 'committed'), all changes should persist on non-volatile storage, such as disks, even if something unexpected happens. Let's break down why this is critical:

Understanding ACID Properties: To fully grasp the concept of durability, we must first recall that ACID stands for Atomicity, Consistency, Isolation, and Durability. Each property plays a crucial role in ensuring reliable database operations.

#### Assumptions
& Context:
The database environment includes transactional processes that modify data state.
A transaction is a unit of work that involves one or more actions performed as an indivisible whole.
First-Principles Reasoning: When a transaction is committed, the database should guarantee that all changes made during this transaction are permanently recorded and cannot be lost due to any unforeseen circumstances.

Let's delve into why durability is crucial:

Atomicity & Durability

Atomicity ensures that each transaction is treated as a single unit of work. If an error occurs during a transaction, the system can roll back (undo) all changes made by that transaction. However, once the transaction commits, these changes must be durable and persist even if there's a subsequent failure.

#### Cons
istency & Durability

#### Cons
istency ensures that transactions move the database from one valid state to another. For example, in a banking application, transferring money between two accounts should maintain a consistent balance. Once such a transfer is committed with durability, it cannot be undone by any later crash or power outage.

Isolation & Durability

Isolation prevents data races and ensures that concurrent transactions do not corrupt the state of the database. Even if multiple transactions are running simultaneously, isolation guarantees their effects will be seen as they were executed in a serial order. Durability is key to ensuring these changes remain consistent even under crash scenarios.

Practical Example: Power Failure Scenario

Imagine a scenario where you transfer $10 from Account A to Account B using a banking application. The transaction involves updating both accounts, subtracting the amount from one and adding it to another. If this transaction is not durable, and there's a power failure right after the subtraction but before the addition, Account A will show a balance of -$10, which is incorrect.

Ensuring Durability

To ensure durability:

Logging: Transactions are logged in a transaction log (often on non-volatile storage) that records all changes.
Checkpointing: Regular checkpoints are taken to ensure that the database can be recovered to a consistent state.
Recovery Procedures: In case of a crash, recovery procedures use the logs and checkpoints to replay transactions or restore the system to a known good state.

#### Common Mistakes
& Follow-Ups
Ignoring Logs: Not maintaining transaction logs can lead to data loss in crashes.
Incomplete Checkpointing: Insufficient checkpointing can result in incomplete recovery, leading to inconsistencies.
Insufficient Recovery Plans: Poorly designed or nonexistent recovery plans can leave the database in a corrupted state post-crash.

In summary, durability is crucial because it ensures that once a transaction commits, all changes are permanently recorded and not lost due to unforeseen interruptions like power failures or system crashes. This property is essential for maintaining data integrity and reliability.

**Comparison:**

|Transaction ID | Status       | Timestamp                |
|--------------|-------------|--------------------------|
|1             | committed   | 2023-10-05T14:30:00Z     |
|Log Entry      | Account ID  | Action    | Amount |
|---------------|------------|-----------|--------|
|1              | 12345       | debit     | 10     |
|2              | 67890       | credit    | 10     |

**Code:**

```sql
-- Example of a transactional SQL statement with logging (hypothetical)
BEGIN TRANSACTION;
UPDATE AccountA SET balance = balance - 10 WHERE id = 12345;
UPDATE AccountB SET balance = balance + 10 WHERE id = 67890;
COMMIT; -- Ensures changes are logged and durable
```

**Example input / output:**

```text
Input:
BEGIN TRANSACTION;
UPDATE AccountA SET balance = balance - 10 WHERE id = 12345;
UPDATE AccountB SET balance = balance + 10 WHERE id = 67890;
COMMIT;

Output:
Transaction [1] committed successfully.
```

**Data:**

```json
{
  "Transaction": {
    "id": 1,
    "status": "committed",
    "timestamp": "2023-10-05T14:30:00Z",
    "log_entries": [
      {"account_id": "12345", "action": "debit", "amount": 10},
      {"account_id": "67890", "action": "credit", "amount": 10}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Transaction starts
4. Is transaction committed?
5. Log changes to non-volatilestorage
6. Commit transaction in memoryand log
7. Raise confirmation to user
8. Rollback uncommittedtransactions
9. Remove log entries for rolledback transactions
10. Transaction successful, datadurable

