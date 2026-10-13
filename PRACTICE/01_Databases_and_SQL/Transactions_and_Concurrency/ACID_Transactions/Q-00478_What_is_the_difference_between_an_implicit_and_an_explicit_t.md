---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between an implicit and an explicit transaction?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Commit Rollback and Savepoints](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Commit_Rollback_and_Savepoints.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database context, transactions are fundamental to maintaining data integrity and consistency. Implicit and explicit transactions are two ways of managing these operations, each with its own characteristics.

#### Triaging the Question

The question asks about the differences between implicit and explicit transactions in databases. The key points the interviewer is probing include:

Understanding of transaction management principles (ACID properties).
Familiarity with how different types of transactions are initiated and managed.
Knowledge of when to use each type of transaction.

#### First-Principles Solution
Implicit Transactions

Implicit transactions, also known as auto-commit or automatic transactions, start and commit automatically without requiring explicit commands. These transactions typically follow the following steps:

Start: A new transaction is started implicitly whenever a query is executed.
Execute: Each statement within the implicit transaction is executed individually.
Commit: If no errors occur during execution, each statement commits automatically.
Explicit Transactions

Explicit transactions require explicit commands to start and commit or rollback. These transactions follow a more controlled process:

Start: A transaction is started with an BEGIN or START TRANSACTION command.
Execute: Multiple statements can be executed within the same transaction context.
Commit/rollback: The transaction ends either by a COMMIT to save changes, or by a ROLLBACK to undo all changes made during the transaction.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, implicit transactions are simpler and more straightforward but less flexible for complex operations that require atomicity. Explicit transactions offer greater control over database state changes, making them suitable for scenarios where multiple operations must be treated as a single unit of work.

#### Common mistakes
include:

Assuming all operations within a query are part of an implicit transaction.
Not committing or rolling back explicit transactions, leading to data inconsistencies.
Overusing explicit transactions without considering the overhead they introduce.

#### Follow-ups
might involve discussing specific use cases for each type or comparing performance implications in different scenarios.

**Code:**

```sql
-- Example of Implicit Transaction
SELECT * FROM users;  -- Statement starts an implicit transaction and is auto-committed

-- Example of Explicit Transaction
BEGIN;
UPDATE users SET balance = balance - 10 WHERE user_id = 1;
UPDATE orders SET status = 'shipped' WHERE order_id = 2;
COMMIT;  -- All changes are committed atomically
```

**Flow:**

1. Implicit transaction startsand executes
2. No
3. Yes
4. Yes
5. No
6. Yes
7. Start Implicit Transaction
8. Execute Query
9. Query Executed
10. Error Occurs?
11. Transaction CommitsAutomatically
12. Transaction Rolls BackAutomatically
13. Start Explicit Transaction
14. Begin Transaction
15. Execute Query 1
16. Execute Query 2?
17. Execute Query 2
18. Error Occurs?
19. Commit Changes
20. Rollback Changes

