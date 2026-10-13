---
type: "practice"
difficulty: "Deep"
---

# What is the effect of a COMMIT on locks held by a transaction?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Transaction Control and Prepared Transactions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Transaction_Control_and_Prepared_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A COMMIT in SQL releases all locks held by a transaction, making the data available to other transactions. However, it also marks the end of the current transaction and makes its changes permanent.

#### Triaging the Question

The question is probing your understanding of how transactions work, specifically focusing on the release of locks when a COMMIT statement is executed. The key aspects to consider are:

Transaction Management: How transactions are started, modified, and ended.
Locks in Databases: What types of locks exist and their purpose.
Commit Statement: The effect of committing a transaction on these locks.

#### Assumptions
We assume the context is a relational database management system (RDBMS) like MySQL or PostgreSQL.
Transactions are started using BEGIN/START TRANSACTION.
Locks can be either row-level or table-level, depending on the isolation level and query operations.
COMMIT ends the transaction by making its changes permanent.
Step-by-Step Solution:

Transaction Start: When you start a transaction with BEGIN or START TRANSACTION, the database engine begins to track changes within that session.

Lock Acquisition: During the execution of queries in the transaction, locks are acquired on data to maintain consistency and prevent concurrent modifications.

Row-Level Locking: E.g., SELECT ... FOR UPDATE.
Table-Level Locking: E.g., UPDATE table_name SET column = value WHERE condition.

Transaction Execution: As you execute DML (Data Manipulation Language) commands like INSERT, UPDATE, or DELETE within the transaction, locks are held on affected data.

Commit Statement:

When a COMMIT is issued, the database releases all locks that were acquired by the transaction.
The changes made during the transaction become permanent and visible to other transactions.
If any errors occur after the BEGIN statement but before the COMMIT, a ROLLBACK can be used to revert all changes.
Detailed Explanation:
Lock Release: A COMMIT releases locks on data, making it available for other transactions. For example, if multiple users are updating the same table, only one user's transaction will succeed due to locking mechanisms.
Transaction End: The transaction is marked as completed and closed. Any changes made within the transaction become part of the database state.
Concurrency Control: This ensures that data modifications do not conflict with concurrent operations.

#### Closing Summary

Understanding how COMMIT works in releasing locks is crucial for efficient database management. It balances between ensuring data integrity and allowing parallel transactions to run smoothly.

#### Common Mistakes/Follow-Ups
Misunderstanding Lock Types: Confusing row-level locks with table-level locks.
Ignoring Deadlocks: Not recognizing the potential for deadlocks when multiple transactions hold conflicting locks.
Transaction Management Errors: Forgetting to ROLLBACK after an error, leading to inconsistent states.

**Code:**

```sql
-- Example of a simple transaction in MySQL
BEGIN;
UPDATE orders SET status = 'processing' WHERE order_id = 123 FOR UPDATE; -- Acquires a lock
COMMIT; -- Releases the lock and makes changes permanent

-- Alternative with explicit COMMIT/ROLLBACK
START TRANSACTION;
UPDATE products SET stock = stock - 1 WHERE product_id = 456 FOR UPDATE;
IF (some condition) THEN
    ROLLBACK; -- Reverts all changes
ELSE
    COMMIT; -- Keeps the transaction changes
END IF;
```

**Example input / output:**

```text
Transaction starts with BEGIN/START TRANSACTION

Input:
BEGIN;
UPDATE products SET stock = stock - 1 WHERE product_id = 456 FOR UPDATE;

Output:
Locks acquired on the row(s) matching product_id = 456.

Input (continued):
COMMIT;

Output:
Locks released, changes made permanent.
```

**Flow:**

1. Yes
2. No
3. Transaction Start withBEGIN/START TRANSACTION
4. Execute Queries
5. Acquire Locks on Data
6. Commit Transaction
7. Release Locks and MakeChanges Permanent
8. Rollback?
9. End of Transaction

