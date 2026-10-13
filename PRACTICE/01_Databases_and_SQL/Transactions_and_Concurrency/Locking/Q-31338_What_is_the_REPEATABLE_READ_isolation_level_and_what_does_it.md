---
type: "practice"
difficulty: "Intermediate"
---

# What is the REPEATABLE READ isolation level and what does it guarantee?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Range Locks and Retries](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Range_Locks_and_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Repeatable Read (RR) is an isolation level used in transactional databases to ensure that a set of data read during a transaction will not change until the transaction completes, even if other transactions modify the same data.

#### Triaging the Question
and Assumptions: The question asks about the Repeatable Read isolation level. This means we need to understand what it does in terms of how concurrent transactions interact with each other when reading data. The primary assumptions here are that you're familiar with basic transactional concepts, such as dirty reads, non-repeatable reads, and phantom reads.

#### First-Principles Solution
Repeatable Read is one of the four standard isolation levels used in databases to manage concurrency. It guarantees a consistency level where:

Read Stability: Once data is read during a transaction, it will remain unchanged until the transaction ends.
No Dirty Reads: A transaction cannot see uncommitted changes made by other transactions.

To understand this better, let's break down how Repeatable Read works from first principles:

1. Transaction Begin

When a transaction begins, it locks all the data that it reads to ensure those values remain unchanged throughout its duration. This locking mechanism prevents any other transaction from modifying or reading these locked rows until the current transaction commits.

2. Data Reading

During the read operations of the transaction, Repeatable Read ensures that once a piece of data is fetched, it stays in its state for the duration of the transaction. Even if another transaction modifies this same data, the original transaction will still see the initial value when it reads again.

3. Handling Concurrency

This isolation level introduces the concept of locking and lock granularity (at different levels like row-level or table-level locks). To manage concurrency efficiently:

Shared Locks: When a transaction starts reading data, it takes out shared (S) locks on those rows.
Exclusive Locks: If a transaction needs to modify the same rows, it will take exclusive (X) locks. Other transactions cannot read or update these locked rows until the lock is released.
4. Transaction Commit and Rollback
Commit: Once all operations within the transaction are completed, any held locks are released.
Rollback: If a transaction fails, all locks taken by it during its execution are immediately removed, ensuring that data remains in a consistent state.

#### Common Mistakes
and Follow-Ups:

Common pitfalls include misunderstanding how Repeatable Read interacts with other isolation levels. For example, it can lead to performance issues if excessive locking occurs, as it might block other transactions from making changes or reading data.

#### Follow-ups
might involve discussing specific scenarios where RR is useful versus less suitable, such as in high-write environments or read-heavy operations.

**Comparison:**

| Isolation Level | Dirty Reads | Non-Repeatable Reads | Phantom Reads |
|-----------------|-------------|---------------------|--------------|
| Read Uncommitted | Yes         | No                  | No           |
| Repeatable Read  | No          | No                  | Yes          |

**Code:**

```sql
-- Example SQL code for setting isolation level and transaction management

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

BEGIN TRANSACTION;

SELECT * FROM Orders WHERE OrderID = 123; -- This will take a shared lock on the row(s) returned

UPDATE Orders SET Status = 'Processed' WHERE OrderID = 123; -- If this statement runs, it will acquire an exclusive lock
```

**Example input / output:**

```text
Input:
Transaction starts with isolation level REPEATABLE READ.
- SELECT * FROM Orders WHERE OrderID = 123 (locks the row)
- UPDATE Orders SET Status = 'Processed' WHERE OrderID = 123 (acquires exclusive lock)

Output:
- After SELECT: Status of OrderID 123 is "Pending"
- After UPDATE: Status of OrderID 123 is changed to "Processed", but other transactions will only see the initial value ("Pending") when they perform a SELECT.
```

**Data:**

```json
{
  "Orders": [
    {
      "OrderID": 123,
      "Status": "Pending"
    },
    {
      "OrderID": 456,
      "Status": "Processed"
    }
  ]
}
```

**Flow:**

1. Transaction Begins
2. Start Reading Data
3. Take Shared Locks on Rows
4. End of Transaction
5. Commit Transaction
6. Release All Locks
7. Update Data
8. Acquire Exclusive Locks
9. Wait for Previous Transactionsto Commit or Rollback
10. Complete Update
11. End of Transaction

