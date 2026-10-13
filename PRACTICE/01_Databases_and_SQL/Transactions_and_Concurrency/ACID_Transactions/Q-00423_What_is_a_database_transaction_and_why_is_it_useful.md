---
type: "practice"
difficulty: "Basic"
---

# What is a database transaction and why is it useful?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ACID Durability and Consistency](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/ACID_Durability_and_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we talk about a database transaction, we're referring to a sequence of operations that are executed as a single unit of work, with a specific guarantee level provided by the ACID properties (Atomicity, Consistency, Isolation, Durability). Transactions ensure that database operations maintain correctness and consistency even in the presence of errors or system failures.

The core idea behind transactions is to provide a way to manage changes to data so that these changes are either completed entirely or not at all, maintaining the integrity of your data. Imagine you're managing a bank account; if there's a transfer from one account to another and something goes wrong (like a network error), you wouldn't want partial funds to be moved.

Let’s break down what each ACID property means:

Atomicity: All operations within a transaction must succeed or fail as a whole. If any part of the transaction fails, all changes are undone.

#### Cons
istency: The transaction moves the database from one valid state to another. No intermediate states should be allowed.
Isolation: Multiple transactions can run concurrently without interfering with each other. This is achieved through locking mechanisms that ensure only one transaction can access a piece of data at any given time.
Durability: Once a transaction is committed, its changes are permanent and survive system failures.
Why Transactions Are Useful
Data Integrity: Ensures that multiple operations on the database are handled as a single unit, maintaining the integrity of your data even if an error occurs during processing.
System Stability: Prevents partial updates and inconsistent states in the database, which could lead to corrupt or erroneous data.
Concurrency Control: Allows concurrent transactions without causing conflicts by managing access to shared resources efficiently.

#### Example Scenario

Imagine you're working on a banking application where a user needs to transfer money from one account to another:

SQL
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - amount WHERE account_id = source_account_id;
UPDATE accounts SET balance = balance + amount WHERE account_id = target_account_id;
COMMIT;

If the second UPDATE fails for any reason, the transaction will be rolled back to its initial state before the transfer was attempted.

#### Common Mistakes
Ignoring Transactions: Failing to use transactions can lead to data inconsistencies if multiple operations are not handled as a single unit.
Overusing Transactions: Excessive use of transactions, especially in simple operations, can increase overhead and reduce performance.
Misunderstanding Isolation Levels: Different isolation levels (Read Uncommitted, Read Committed, Repeatable Read, Serializable) have different trade-offs between performance and consistency. Choosing the wrong level can lead to issues like dirty reads or phantom reads.

**Code:**

```sql
BEGIN TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 12345;
COMMIT; -- or ROLLBACK if something goes wrong.
```

