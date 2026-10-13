---
type: "practice"
difficulty: "Intermediate"
---

# How would you write a transaction to transfer money from account A to account B safely?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Atomicity and Money Transfers](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Atomicity_and_Money_Transfers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure safe and accurate transfers of funds between two accounts, we need to leverage transactions to guarantee atomicity, consistency, isolation, and durability (ACID properties). This means that the transfer operation must be treated as a single unit of work where both the debiting from account A and crediting to account B are performed atomically. If any part of this operation fails, neither should occur, ensuring data integrity.

To write such a transaction in SQL, we first need to understand the operations involved: reading the balance of each account, updating their balances with the transfer amount, and finally committing or rolling back the changes based on whether all steps succeed.

**Comparison:**

```
Account ID	Balance
account_A	100
account_B	50

Final state after successful transaction:

Account ID	Balance
account_A	80
account_B	70
```

**Code:**

```sql
START TRANSACTION;

-- Step 1: Read the current balance of both accounts
SELECT balance INTO @balanceA FROM accounts WHERE id = 'account_A';
SELECT balance INTO @balanceB FROM accounts WHERE id = 'account_B';

-- Step 2: Check if there are sufficient funds in account A to perform the transfer
IF @balanceA >= amount THEN
    -- Step 3: Update the balances
    UPDATE accounts SET balance = balance - amount WHERE id = 'account_A';
    UPDATE accounts SET balance = balance + amount WHERE id = 'account_B';

    -- Step 4: Commit the transaction if all steps succeed
    COMMIT;
ELSE
    -- Step 5: Rollback the transaction if there are insufficient funds in account A
    ROLLBACK;
END IF;
```

**Example input / output:**

```

Input:

amount: 20 Output (after successful transaction):
JSON
Copy
{
  "accounts": [
    { "id": "account_A", "balance": 80 },
    { "id": "account_B", "balance": 70 }
  ]
}

Failed Output (due to insufficient funds in account_A):

JSON
Copy
No changes made to the accounts.
```

**Data:**

```json
{
  "accounts": [
    { "id": "account_A", "balance": 100 },
    { "id": "account_B", "balance": 50 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Check if balance of A >=amount
5. (Yes)
6. Update balances: A -= amount,B += amount
7. Commit transaction
8. End
9. (No)
10. Rollback transaction

