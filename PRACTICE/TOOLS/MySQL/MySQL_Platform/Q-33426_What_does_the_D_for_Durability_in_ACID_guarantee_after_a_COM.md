---
type: "practice"
difficulty: "Basic"
---

# What does the 'D' for Durability in ACID guarantee after a COMMIT?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Durability is one of the four cornerstones that make up the ACID properties (Atomicity, Consistency, Isolation, and Durability). In MySQL, the 'D' in ACID guarantees that once a transaction has been committed, all changes made within it are permanently saved to the database. This means that even if there is a system failure or power outage right after the commit, the data modifications will still be intact.

To understand this better, let's break down what Durability ensures step by step:

Transaction Completion: When you issue a COMMIT command, all operations within that transaction are considered complete and irreversible.
Persistent Storage: The changes made during the transaction are written to the database’s permanent storage (e.g., on disk), not just in memory or temporary files.
Recovery Mechanisms: MySQL employs various mechanisms like binary logging and transaction logs to ensure data integrity, even if a failure occurs after the COMMIT.

Now let's reason through this from first principles:

Transaction State: Initially, when you start a transaction using BEGIN or START TRANSACTION, no changes are visible outside the transaction.
Changes Within Transaction: You perform various SQL operations like inserts, updates, and deletes within the transaction scope.
Commit Command: When you issue a COMMIT command, MySQL ensures that all changes made in the transaction are written to the disk before allowing any further commands.

Here’s an example scenario:

SQL
START TRANSACTION;
UPDATE orders SET status = 'Shipped' WHERE order_id = 12345;
-- Assume some network or system failure occurs here.
COMMIT; -- Even if there's a crash right after this, changes will be saved.

In the above case, even if the system crashes immediately after the COMMIT, the update to the status field of the order with ID 12345 would still be persisted in the database.

Let’s summarize and highlight common mistakes:

Mistake: Assuming that changes are only stored in memory. In MySQL, a COMMIT ensures these changes are written to the disk.
Mistake: Thinking that a crash at any point during or right after a transaction can undo committed changes.

In conclusion, Durability in ACID guarantees that once a transaction is committed, all its modifications will be permanently saved to the database's durable storage. This ensures data integrity and consistency even in the face of unexpected failures.

**Code:**

```sql
START TRANSACTION;
UPDATE orders SET status = 'Shipped' WHERE order_id = 12345;
COMMIT; -- Changes are guaranteed to be written to disk after this command.
```

**Flow:**

1. Yes
2. No
3. Start Transaction
4. Perform Operations
5. Is Commit Issued?
6. Write to Disk
7. Commit Successful
8. Continue Transaction

