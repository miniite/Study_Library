---
type: "practice"
difficulty: "Intermediate"
---

# What is a shared lock versus an exclusive lock?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Lock Holding and Optimistic Concurrency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Lock_Holding_and_Optimistic_Concurrency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Shared locks and exclusive locks are fundamental concepts in database systems, particularly when it comes to transaction management and concurrency control. In simple terms, a shared lock (S-lock) allows multiple transactions to read data simultaneously but prevents any other transaction from modifying the data until the lock is released. On the other hand, an exclusive lock (X-lock) ensures that only one transaction can access the data for reading or writing at any given time.

To fully understand these concepts, let's break down their definitions and explore how they work in practice.

**Comparison:**

| Transaction | Lock Type | Table Name | Affected Rows |
|-------------|-----------|------------|---------------|
| 1           | S         | products   | 5             |
| 2           | S         | products   | 3             |
| 3           | S         | customers  | 4             |

**Code:**

```
MERMAID
Copy
flowchart TD
    A[Transaction Starts] --> B[Reads Data with S-lock]
    B --> C[Other Transactions Can Read but Not Write]
    C --> D{Does Any Transaction Want to Write?}
    D -- Yes --> E[Exclusive Lock Acquired for Writing]
    E --> F[Conflict with S-locks; Rollback or Wait]
    D -- No --> G[Release S-lock and End Transaction]
```

**Example input / output:**

```text
Transaction 1:
- Acquires S-lock on table 'products'
- Reads and updates some rows

Transaction 2:
- Tries to acquire X-lock for writing
- Waits because Transaction 1 holds an S-lock
- Release the S-lock once finished reading

Transaction 3:
- Tries to read data from 'products'
- Acquires S-lock as Transaction 2 is still waiting
```

**Data:**

```json
{
  "data": [
    {
      "transactionId": 1,
      "lockType": "S",
      "tableName": "products",
      "affectedRows": 5
    },
    {
      "transactionId": 2,
      "lockType": "S",
      "tableName": "products",
      "affectedRows": 3
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Transaction Starts
6. Reads Data with S-lock
7. Other Transactions Can Readbut Not Write
8. Does Any Transaction Want toWrite?
9. Exclusive Lock Acquired forWriting
10. Conflict with S-locks; Rollbackor Wait
11. Release S-lock and EndTransaction
12. Transaction Starts
13. Reads Data with X-lock
14. Does Any Other TransactionHold an S-lock?
15. Wait for S-locks to Release
16. Exclusive Lock Acquired andModify Data

