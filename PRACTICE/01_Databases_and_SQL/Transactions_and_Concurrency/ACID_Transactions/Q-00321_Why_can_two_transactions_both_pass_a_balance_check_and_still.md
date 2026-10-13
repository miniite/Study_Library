---
type: "practice"
difficulty: "Deep"
---

# Why can two transactions both pass a balance check and still overdraw an account under READ COMMITTED, and how do you close that hole?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Atomicity and Money Transfers](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Atomicity_and_Money_Transfers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why two transactions can both pass a balance check and still overdraw an account under READ COMMITTED, we need to dive into the core concepts of ACID properties, particularly focusing on how they interact with concurrent operations in a database.

#### Triaging the Question

The question is essentially probing our understanding of:

The READ COMMITTED isolation level.
How race conditions can occur even when transactions appear to pass certain checks.
Techniques and strategies to prevent such overdraw issues.

#### Assumptions
We are dealing with a database that uses the READ COMMITTED isolation level.
Two concurrent transactions are accessing the same account balance.
The system performs a balance check before attempting to debit or credit an account.
Step-by-Step Solution

Understanding READ COMMITTED: In READ COMMITTED, a transaction sees only data committed prior to the start of its own transaction, and not changes made by other transactions that have committed since it began. This means the transaction sees a snapshot of the database at the moment it starts.

Race Condition Scenario:

Transaction A begins.
The system checks the current balance (say, 100).
Before committing any change, Transaction B begins.
Transaction B checks the same balance (still showing 100) and decides to debit 50.
Transaction B commits its changes, reducing the balance to 50.
Transaction A then tries to credit 50 but sees the new lower balance of 50. It correctly checks that there are sufficient funds before committing.

The Overdraw Issue: Even though both transactions check and see a consistent snapshot (100), they can still overdraw because:

The system only checks at the moment each transaction starts.
Between the start of one transaction and its completion, other changes might occur that are not visible to it.

Closing the Hole: To prevent this issue, we need a mechanism that ensures transactions see a consistent view of the database throughout their execution, or use a higher isolation level like SERIALIZABLE.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key takeaway is that READ COMMITTED can lead to race conditions where concurrent transactions might incorrectly believe they have sufficient funds based on an outdated snapshot. To address this:

Use REPEATABLE READ, which locks the row(s) read by a transaction until it ends, ensuring no other transaction can change them.

#### Cons
ider using SERIALIZABLE if absolute consistency is required.

#### Common mistakes
include assuming that simply checking balances at the start of transactions will prevent all overdraw issues. Always consider the entire life cycle of each transaction and potential race conditions.

**Comparison:**

```
Account ID	Balance
101	100
```

**Code:**

```sql
-- Example SQL to illustrate the issue
BEGIN TRANSACTION;

SELECT balance FROM accounts WHERE account_id = 101 FOR UPDATE; -- Locks the row

IF (balance >= 50) THEN
    UPDATE accounts SET balance = balance - 50 WHERE account_id = 101;
END IF;

COMMIT;
```

**Example input / output:**

```text
- **Input**: Two transactions attempting to debit $50 from the same account.
- **Output**: Both transactions pass the initial balance check but can overdraw because they do not lock rows or use a higher isolation level.
```

**Data:**

```json
{
  "account_id": 101,
  "balance": 100
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Transaction A starts
6. Check balance
7. Read current balance (100)
8. Debit $50?
9. Update balance to 50
10. Commit transaction
11. Transaction B starts
12. Check balance
13. Read current balance (100)
14. Debit $50?
15. Update balance to 50
16. Commit transaction

