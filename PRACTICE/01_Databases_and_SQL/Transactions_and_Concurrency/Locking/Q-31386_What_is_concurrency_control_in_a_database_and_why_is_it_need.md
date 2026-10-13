---
type: "practice"
difficulty: "Basic"
---

# What is concurrency control in a database and why is it needed?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MVCC](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/MVCC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Concurrency control in a database refers to mechanisms that ensure data consistency and integrity when multiple transactions access and modify the same data concurrently. It's crucial because if not managed properly, concurrent transactions can lead to issues like dirty reads, non-repeatable reads, or phantom reads. Essentially, concurrency control helps maintain the ACID properties (Atomicity, Consistency, Isolation, Durability) of a database by preventing conflicts during read and write operations.

**Code:**

```sql
-- Example SQL statement for a transaction that may conflict with another transaction
BEGIN TRANSACTION;
SELECT balance FROM account WHERE id = 1 FOR UPDATE; -- Locks the row until the transaction is complete
UPDATE account SET balance = balance - 50 WHERE id = 1;
COMMIT;
```

**Example input / output:**

```text
Input:
- Transaction T1: `SELECT balance FROM account WHERE id = 1 FOR UPDATE;`
- Transaction T2: `SELECT balance FROM account WHERE id = 2 FOR UPDATE;`
Output:
Both transactions successfully acquire locks on their respective rows and can proceed without conflict. If a third transaction attempts to modify the same row, it will have to wait until one of the transactions releases its lock.
```

**Data:**

```json
{
  "account": [
    { "id": 1, "balance": 1000 },
    { "id": 2, "balance": 800 }
  ]
}
```

