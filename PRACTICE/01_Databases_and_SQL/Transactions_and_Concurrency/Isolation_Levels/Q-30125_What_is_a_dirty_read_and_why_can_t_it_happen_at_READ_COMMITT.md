---
type: "practice"
difficulty: "Basic"
---

# What is a dirty read and why can't it happen at READ COMMITTED or above? Give a scenario where it would corrupt a financial report.

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Phantoms and Double Booking](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Isolation_Levels/Phantoms_and_Double_Booking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Dirty reads occur when one transaction can see uncommitted changes made by another concurrent transaction, leading to inconsistent data visibility. At the READ COMMITTED isolation level and above (including REPEATABLE READ and SERIALIZABLE), dirty reads are prevented because transactions only read committed data.

To understand why dirty reads cannot happen at READ COMMITTED or higher isolation levels, let's first define each of these levels:

READ UNCOMMITTED: Allows a transaction to see all changes made by other concurrent transactions, including uncommitted ones. This is the least restrictive level and can lead to dirty reads.
READ COMMITTED: Ensures that only committed data is visible, so a transaction cannot read uncommitted changes from another transaction.
REPEATABLE READ: Guarantees that once a row has been retrieved by a transaction, repeated access during the same transaction will see the same data. It achieves this by locking rows as they are accessed, preventing other transactions from modifying them until the current transaction is finished.
SERIALIZABLE: Provides the highest level of isolation, ensuring that transactions execute serially, one after another, mimicking a single-threaded environment.

Let's illustrate these levels with an example involving a financial report:

Scenario

Suppose we have a table FinancialReport with a column Balance, and two transactions (T1 and T2) running concurrently. The initial balance is 100 units.

Transaction T1:

Reads the current balance: 100.
Modifies the balance to 90 for reporting purposes, but does not commit this change yet.

Transaction T2:

Reads the balance as seen in T1: 90 (uncommitted change).
Uses this uncommitted value of 90 in a report and commits it.

In a system with READ UNCOMMITTED, the dirty read occurs because T2 sees the uncommitted balance from T1. However, at READ COMMITTED and above:

READ COMMITTED: Ensures that T2 only reads the committed balance (100), preventing it from seeing T1's uncommitted change.
REPEATABLE READ: Locks the row when read by T2, preventing any changes by T1. Thus, T2 always sees 100.
SERIALIZABLE: Serializes transactions so that they run one after another. Since T1 has not committed its change yet, it will block T2 from reading the uncommitted value.
Why Dirty Reads Can't Happen at READ COMMITTED or Above

At these isolation levels, the database ensures data consistency by either blocking uncommitted changes (as in REPEATABLE READ) or ensuring that transactions only see committed data (as in READ COMMITTED). This prevents T2 from reading an uncommitted balance of 90 and using it for reporting purposes.

#### Common Mistakes
Misunderstanding Isolation Levels: Thinking that READ UNCOMMITTED is the default setting, which can lead to dirty reads.
Not Considering Locking Mechanisms: Assuming that transactions will always see the latest data without considering lock acquisition strategies.
Ignoring Data Consistency Requirements: Overlooking the importance of ensuring that a transaction sees consistent and reliable data.

#### Closing Summary

In summary, at READ COMMITTED or above isolation levels, dirty reads are prevented because only committed data is visible. This ensures data consistency and accuracy in operations like financial reporting. Understanding these isolation levels helps in designing robust database systems that can handle concurrent transactions without corrupting data.

**Code:**

```sql
-- Example SQL statements to illustrate READ COMMITTED behavior

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

BEGIN TRANSACTION;

SELECT Balance FROM FinancialReport; -- Reads committed balance (100)
UPDATE FinancialReport SET Balance = 90 FOR UPDATE; -- Locks the row for update but does not commit yet

COMMIT;
```

**Example input / output:**

```

Input:

SQL
Copy
BEGIN TRANSACTION;

-- Transaction T2 reads balance at READ COMMITTED level (committed balance is read)
SELECT Balance FROM FinancialReport FOR UPDATE;

Output:

PLAINTEXT
Copy
+--------+
| Balance|
+--------+
|      100|
+--------+
No uncommitted change is visible.
```

**Data:**

```json
{
  "FinancialReport": {
    "id": 1,
    "balance": 100
  }
}
```

