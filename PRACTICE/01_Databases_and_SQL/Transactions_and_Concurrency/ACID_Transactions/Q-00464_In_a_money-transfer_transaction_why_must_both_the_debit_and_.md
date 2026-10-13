---
type: "practice"
difficulty: "Basic"
---

# In a money-transfer transaction why must both the debit and credit succeed or both fail?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Atomicity and Money Transfers](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Atomicity_and_Money_Transfers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing transactions in databases, ensuring that all parts of a transaction either completely succeed or entirely fail is crucial to maintain consistency and integrity. In a money-transfer transaction between two accounts, both the debit from one account (let's call it Account A) and the credit to another account (Account B) must succeed or fail together. This is often ensured through Atomicity, Consistency, Isolation, and Durability (ACID) properties of transactions.

Firstly, let’s break down why ACID properties are essential in this context:

Atomicity: The transaction should be treated as a single unit of work; if any part fails, the entire transaction must be rolled back.

#### Cons
istency: The database moves from one valid state to another. A money-transfer that partially succeeds could leave both accounts in an inconsistent state, where Account A has less but not all funds subtracted and Account B has more than expected.
Isolation: This prevents concurrent transactions from interfering with each other. Without proper isolation, a transfer might be seen by one transaction before it is fully committed, leading to incorrect balance calculations.
Durability: Once a transaction is committed, its effects are permanent. A failed transaction should ensure no partial updates are stored.

Now, let’s consider the scenario where a money-transfer transaction from Account A to Account B fails partially:

Account A Debit Succeeds, Credit Fails:

If only the debit from Account A succeeds but the credit to Account B fails, Account A will have less funds than it should, and Account B will still have its original balance.
This inconsistency can lead to incorrect accounting records and financial discrepancies.

Account B Credit Succeeds, Debit Fails:

Conversely, if only the credit to Account B succeeds but the debit from Account A fails, Account B will now have an extra amount of money that should not be there.
This again creates an inconsistent state in the database and can cause financial issues.

To ensure atomicity and consistency are maintained, both operations must succeed or fail together. This is typically achieved through explicit transaction handling:

SQL
START TRANSACTION;
-- Debit from Account A
UPDATE accounts SET balance = balance - transfer_amount WHERE account_id = A;

-- Credit to Account B
UPDATE accounts SET balance = balance + transfer_amount WHERE account_id = B;

COMMIT; -- Only commit if both updates are successful

Or, in the case of a failure:

SQL
ROLLBACK; -- Rollback all changes if any part fails

**Comparison:**

```
Transaction State	Account A Balance	Account B Balance
Before Transfer	500	300
After Successful	400	400
After Failed Debit	500	300
```

**Code:**

```sql
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 'A';
UPDATE accounts SET balance = balance + 100 WHERE account_id = 'B';
COMMIT;

-- In case of failure, rollback the changes
ROLLBACK;
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Start Transaction<br>Debit from Account A (500 - 100 = 400)<br>Credit to Account B (300 + 100 = 400)<br>Commit Transactions | Both accounts updated correctly: A: 400, B: 400 |
| Start Transaction<br>Debit from Account A (500 - 100 = 400) <br>Credit to Account B (300 + 100 = 400)<br>Simulate Debit Failure | Rollback Transactions<br>Accounts revert to original balances: A: 500, B: 300 |

**Data:**

```json
{
    "accounts": [
        {"account_id": "A", "balance": 500},
        {"account_id": "B", "balance": 300}
    ]
}
```

**Flow:**

1. Start Transaction
2. Debit from Account A
3. Credit to Account B
4. Commit Transactions
5. Both Accounts UpdatedCorrectly
6. Simulate Debit Failure
7. Rollback Transactions
8. Accounts Revert to OriginalBalances

