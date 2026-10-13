---
type: "practice"
difficulty: "Deep"
---

# You need account balance checks to never overdraft under concurrency. Compare SELECT FOR UPDATE at READ COMMITTED versus running at SERIALIZABLE.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lost Updates and Write Skew](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Lost_Updates_and_Write_Skew.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure account balance checks never result in overdrafts under concurrent transactions, we must carefully consider and compare two isolation levels: READ COMMITTED with SELECT FOR UPDATE and SERIALIZABLE. In this context, READ COMMITTED is a basic transaction isolation level where each transaction sees the state of data as it was at the start of its execution. Meanwhile, SERIALIZABLE ensures that transactions see an isolated snapshot of the database, similar to running in separate serial order.

The core difference lies in how these levels handle concurrent read and write operations on shared data. Let’s explore this in detail:

#### Assumptions

We are working with a banking system where account balances need to be checked and potentially updated.
Multiple transactions can concurrently access and modify the same accounts.
The primary concern is preventing overdrafts.

READ COMMITTED vs SELECT FOR UPDATE:

READ COMMITTED with SELECT FOR UPDATE
In this scenario, each transaction sees the latest committed state of the database at the start of its execution.
However, if a transaction locks a row using SELECT FOR UPDATE, it will prevent other transactions from modifying that specific row until the lock is released.
This can lead to scenarios where two concurrent transactions might read different states of the data due to the non-repeatable reads allowed by READ COMMITTED.

#### Example Scenario

Transaction T1 starts at 9:00 AM and runs for 5 minutes, reading a balance of $100 from account A.
Transaction T2 starts at 9:02 AM and also needs to read the same account A. At this point, T1 has not yet committed or updated the balance.
T2 reads the balance as still being $100 (since it sees the state at 9:02), but in reality, the balance might have been updated by T1 before committing due to network latency or transaction delay.
SERIALIZABLE
In a SERIALIZABLE isolation level, transactions see data as if they were executed one after another, serially. This prevents non-repeatable reads and phantom reads.
To implement this, each read operation acquires locks that prevent other transactions from modifying the same rows until the current transaction commits or rolls back.

#### Example Scenario

Transaction T1 starts at 9:00 AM, locks account A with SELECT FOR UPDATE, and reads a balance of $100.
Transaction T2 starts at 9:02 AM and tries to lock account A. Since the row is already locked by T1, T2 will wait until T1 commits or rolls back.
Only after T1 either commits or rolls back can T2 proceed with its read operation.
Comparison in Detail:
Concurrency: SERIALIZABLE inherently limits concurrency due to strict locking mechanisms, which could lead to bottlenecks and increased contention for resources.
Read Consistency: SERIALIZABLE ensures that no transaction reads "dirty" data (uncommitted changes from other transactions), making it safer but potentially more resource-intensive.

#### Conclusion
Given the requirement to prevent overdrafts, SERIALIZABLE is the preferred isolation level because it ensures consistent and repeatable reads, avoiding scenarios where a transaction might read stale data leading to potential overdrafts.
However, READ COMMITTED with SELECT FOR UPDATE can still be used effectively if implemented correctly, by carefully managing lock acquisition and release times.

**Comparison:**

| Transaction Isolation Level | Read Consistency       | Concurrency           |
|-----------------------------|------------------------|-----------------------|
| READ COMMITTED              | Non-repeatable         | Higher                |
| SERIALIZABLE                | Repeatable             | Lower                 |

**Code:**

```sql
-- Example of using SERIALIZABLE isolation level
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
SELECT * FROM accounts WHERE account_id = 12345 FOR UPDATE;
-- Perform operations or further checks
COMMIT;

-- Example of READ COMMITTED with SELECT FOR UPDATE
BEGIN TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM accounts WHERE account_id = 12345 FOR UPDATE; -- Locks the row until commit/rollback
-- Perform operations or further checks
COMMIT;
```

**Example input / output:**

```text
Input:
- Start transaction with isolation level SERIALIZABLE.
- Execute SELECT * FROM accounts WHERE account_id = 12345 FOR UPDATE.

Output:
- Locks row for account 12345 and reads balance 100.
- Perform operations or further checks.
- Commit transaction.

Input:
- Start transaction with isolation level READ COMMITTED.
- Execute SELECT * FROM accounts WHERE account_id = 12345 FOR UPDATE; (locks the row until commit/rollback)

Output:
- Locks row for account 12345 and reads balance 100.
- Perform operations or further checks.
- Commit transaction.
```

**Data:**

```json
{
  "accounts": [
    {
      "account_id": 12345,
      "balance": 100
    }
  ]
}
```

**Flow:**

1. READ COMMITTED
2. SERIALIZABLE
3. Start Transaction
4. Isolation Level?
5. Read & Lock Row
6. Perform Operations
7. Commit or Rollback
8. Lock and Read Row
9. Perform Operations
10. Commit or Rollback

