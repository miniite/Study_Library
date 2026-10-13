---
type: "practice"
difficulty: "Basic"
---

# Give a real-world example where a transaction is necessary such as a money transfer between two accounts.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Atomicity and Money Transfers](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Atomicity_and_Money_Transfers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a financial system, ensuring that a money transfer from one account to another happens correctly and consistently is crucial. This task requires transactions because it involves multiple steps that must be completed atomically—either all of them succeed or none do. A transaction ensures data integrity and consistency by adhering to the ACID properties: Atomicity, Consistency, Isolation, and Durability.

#### Triaging the Question

The question is probing your understanding of how transactions maintain data integrity in a scenario where multiple related operations need to be performed simultaneously. Key points include:

Atomicity: All steps must succeed or none should.

#### Cons
istency: The database transitions from one valid state to another, without leaving it in an inconsistent state.
Isolation: Operations by different transactions are isolated from each other; they do not interfere with each other's execution.

#### First-Principles Solution

Let's consider a money transfer scenario between two accounts: Account A (with $100) and Account B (with $50). The transaction should ensure that:

Account A debits $50.
Account B credits $50.
Both operations must either succeed or fail together.
Step-by-Step Solution
Start a Transaction:
Begin the transaction to group the operations.
Debit Account A:
Check if the balance of Account A is sufficient (at least $50).
If yes, reduce the balance by $50; otherwise, raise an error and rollback the transaction.
Credit Account B:
Increase the balance of Account B by $50.
Commit Transaction:
If all operations are successful, commit the transaction to make changes permanent.
Rollback on Failure:
If any step fails, roll back the transaction to undo all changes and maintain data integrity.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In this example, transactions ensure that either both operations (debiting and crediting) are completed successfully or neither is. This prevents situations like Account A being debited but not having the funds credited elsewhere, leading to inconsistent states.

#### Common Mistakes

Failing to start a transaction before performing multiple operations.
Not checking the balance before attempting a debit operation.
Ignoring proper error handling and rollback mechanisms.

#### Follow-Ups

How would you handle concurrent transactions in this scenario?
What about dealing with large amounts of money and performance considerations?

**Comparison:**

```
Account ID	Balance Before	Balance After (Successful)	Balance After (Failed)
A	100	50	100
B	50	100	50
```

**Code:**

```sql
BEGIN TRANSACTION;

-- Step 2: Debit Account A
UPDATE accounts SET balance = balance - 50 WHERE account_id = 'A';
IF @@ROWCOUNT = 0 -- Check if the update was successful
    ROLLBACK;
ELSE
    -- Step 3: Credit Account B
    UPDATE accounts SET balance = balance + 50 WHERE account_id = 'B';

    -- Step 4: Commit Transaction
    COMMIT;
END;
```

**Example input / output:**

```

Input (Before Transaction):

JSON
Copy
{
  "accounts": [
    {"account_id": "A", "balance": 100},
    {"account_id": "B", "balance": 50}
  ]
}

Output (After Successful Transaction):

JSON
Copy
{
  "accounts": [
    {"account_id": "A", "balance": 50},
    {"account_id": "B", "balance": 100}
  ]
}

Output (After Failed Transaction - Insufficient Funds in Account A):

JSON
Copy
{
  "accounts": [
    {"account_id": "A", "balance": 100}, // No change since the transaction was rolled back
    {"account_id": "B", "balance": 50}   // No change since the transaction was rolled back
  ]
}
```

**Data:**

```json
{
  "accounts": [
    {"account_id": "A", "balance": 100},
    {"account_id": "B", "balance": 50}
  ]
}
```

**Flow:**

1. Insufficient Funds
2. Sufficient Funds
3. Start Transaction
4. Debit Account A
5. Check Balance
6. Rollback and End
7. Credit Account B
8. Commit Transaction
9. End

