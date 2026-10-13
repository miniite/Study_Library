---
type: "practice"
difficulty: "Basic"
---

# What is a transaction? Explain BEGIN, COMMIT, and ROLLBACK.

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ DELETE TRUNCATE and DROP](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/DELETE_TRUNCATE_and_DROP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, a transaction is a sequence of database operations that are treated as a single unit of work. Each transaction must be atomic (all or nothing), consistent (it leaves the database in a valid state), isolated (it does not interfere with other transactions), and durable (once committed, it remains so even if there are failures).

To manage these transactions, PostgreSQL provides three key commands: BEGIN, COMMIT, and ROLLBACK. Let's break down each of these:

#### Triage
What the Question is Asking: The question asks for an explanation of transaction management in PostgreSQL, specifically focusing on how to start a new transaction (BEGIN), complete it successfully (COMMIT), or undo changes if something goes wrong (ROLLBACK).
Key Assumptions: We assume you are familiar with basic SQL operations and understand the importance of transactions in maintaining data integrity.

#### First Principles Solution
BEGIN: This command marks the start of a new transaction. All subsequent queries until the COMMIT or ROLLBACK will be part of this transaction.
COMMIT: Once you are satisfied with your changes, use COMMIT. This command ensures that all transactions within it are completed and saved permanently to the database.
ROLLBACK: If an error occurs during a transaction, or if you decide not to save the current state, use ROLLBACK. All changes made within the transaction are undone.

#### Example

Let's look at how these commands work in practice:

SQL
-- Start a new transaction
BEGIN;

-- Perform some operations
INSERT INTO customers (name, email) VALUES ('John Doe', 'john@example.com');

-- If an error occurs here or if you want to undo the changes
ROLLBACK;

-- Or, if everything is fine, commit the changes
COMMIT;

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary:

BEGIN starts a transaction.
COMMIT finalizes and saves all transactions within it.
ROLLBACK undoes any operations performed during the current transaction.

#### Common mistakes
include not using BEGIN to define a transaction, forgetting to use COMMIT or ROLLBACK, which can lead to incomplete changes being applied or data corruption. It's also important to ensure that your queries are atomic and idempotent within a transaction to maintain consistency.

Common Follow-Ups
How do transactions interact with concurrency?
What happens if a query fails during a transaction?
Can you explain how PostgreSQL handles locking in transactions?

**Code:**

```sql
BEGIN;
INSERT INTO customers (name, email) VALUES ('John Doe', 'john@example.com');
ROLLBACK;  -- or COMMIT;
```

