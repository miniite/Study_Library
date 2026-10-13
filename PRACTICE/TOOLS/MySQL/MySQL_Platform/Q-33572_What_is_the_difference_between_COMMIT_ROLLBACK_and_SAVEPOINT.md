---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between COMMIT, ROLLBACK, and SAVEPOINT?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Online Schema Changes](<../../../../TOOLS/MySQL/MySQL_Platform/Online_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with transactions in MySQL, COMMIT, ROLLBACK, and SAVEPOINT are essential commands used to manage the state of database operations. In a nutshell:

COMMIT: Saves all changes made within a transaction.
ROLLBACK: Reverts all changes made within a transaction back to their original state before the transaction started.
SAVEPOINT: Creates an intermediate point (savepoint) within a transaction, allowing for selective rollbacks.

Let's dive into each of these concepts and understand how they work in detail.

**Code:**

```sql
-- Example transaction with savepoints

START TRANSACTION;

INSERT INTO users (name, email) VALUES ('John Doe', 'john@example.com');

SAVEPOINT sp1;

UPDATE users SET email = 'johndoe@example.com' WHERE name = 'John Doe';

SAVEPOINT sp2;

DELETE FROM users WHERE name = 'John Doe';

COMMIT;  -- Saves all changes

ROLLBACK TO SAVEPOINT sp1;  -- Reverts to the state after the first savepoint
```

**Flow:**

1. Savepoint sp1
2. Savepoint sp2
3. Transaction Begins
4. Insert into Users
5. Savepoint Created
6. Update Users Email
7. Savepoint Created
8. Delete from Users
9. Commit Saves All Changes
10. Transaction Ends
11. Rollback to Savepoint sp1
12. Revert Insert and Update, KeepDelete Unexecuted

