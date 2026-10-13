---
type: "practice"
difficulty: "Intermediate"
---

# What happens if the application crashes after the debit but before the credit in a transfer?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Atomicity and Money Transfers](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Atomicity_and_Money_Transfers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an application performs a database transaction to move funds from one account to another, it's crucial that both steps (debit and credit) are completed successfully or neither is. This ensures consistency across the accounts, adhering to ACID properties—Atomicity, Consistency, Isolation, Durability. If the application crashes after debiting but before crediting, we need to ensure that the transaction is rolled back to maintain consistency.

The question asks what happens in such a scenario and is probing our understanding of how database transactions handle failures and rollbacks. We'll assume the database uses ACID-compliant transactions.

**Comparison:**

```
Account ID	Initial Balance	Updated Balance
source_account_id	500	400
destination_account_id	300	300
```

**Code:**

```sql
-- Example SQL for a transfer between accounts
BEGIN TRANSACTION;

-- Debit source account
UPDATE accounts SET balance = balance - 100 WHERE id = 'source_account_id';

-- Credit destination account
UPDATE accounts SET balance = balance + 100 WHERE id = 'destination_account_id';

COMMIT;
```

**Example input / output:**

```

Initial State:

source_account_id balance: $500
destination_account_id balance: $300

Transaction Attempt (Assuming the transaction is within a single database session):

SQL
Copy
BEGIN TRANSACTION;

-- Debit source account
UPDATE accounts SET balance = 400 WHERE id = 'source_account_id';

-- Credit destination account
UPDATE accounts SET balance = 400 WHERE id = 'destination_account_id';

-- This point is where the application crashes

Outcome:

source_account_id balance: $400 (unchanged because of transaction failure)
destination_account_id balance: $300 (unchanged because of transaction failure)

The system should ensure that any partially completed transactions are rolled back, leaving the database in a consistent state.
```

**Data:**

```json
{
  "accounts": [
    {
      "id": "source_account_id",
      "balance": 500
    },
    {
      "id": "destination_account_id",
      "balance": 300
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Transaction Starts
4. Application Completes BothSteps?
5. Commit Transaction
6. Rollback Transaction
7. Both Accounts RemainConsistent

