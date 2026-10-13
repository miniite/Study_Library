---
type: "practice"
difficulty: "Intermediate"
---

# What is a dirty read and which isolation level allows it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Isolation Levels in Transactions](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/ACID_Transactions/Isolation_Levels_in_Transactions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Dirty reads, also known as non-repeatable reads or unrepeatable reads, occur in database systems when a transaction reads data that has been modified but not yet committed by another transaction. In other words, if one transaction can read uncommitted (dirty) data from another transaction, it results in a dirty read.

In the context of isolation levels, a dirty read is allowed at the lowest isolation level: Read Uncommitted. This level permits transactions to see uncommitted changes made by other transactions, which can lead to inconsistencies and non-deterministic behavior if not handled carefully.

Let's break down the concept from first principles:

Understanding Isolation Levels:

Read Committed: Ensures that a transaction cannot read data modified in uncommitted transactions.
Repeatable Read: Ensures that multiple reads within a single transaction return the same results, even if other transactions modify and commit data between those reads.
Serializable: The strongest isolation level, ensuring that transactions are completely isolated from each other. It guarantees that every transaction sees the database in a state that is consistent with all committed transactions.

Read Uncommitted:

In this isolation level, any uncommitted changes made by another transaction can be read.
This can introduce inconsistencies because a read operation might see data that could later be rolled back due to some error or decision by the other transaction.

#### Example Scenario

Suppose two transactions, T1 and T2, are running in a database with isolation level set to Read Uncommitted.
T1 starts and reads a value from a table.
Before T1 commits its changes, T2 modifies the same row.
When T1 later attempts to read the same row again, it sees the modified (dirty) data.

Why Use Read Uncommitted?

Performance improvement: By avoiding locking and waiting for other transactions to commit, performance can be boosted, especially in scenarios with high concurrency but where consistency is less critical.

#### Trade-offs
Increased risk of dirty reads, which may lead to logical inconsistencies or incorrect data processing.

#### Closing Summary
Dirty reads are a consequence of the lowest isolation level, Read Uncommitted, where transactions can read uncommitted changes made by other transactions. While this level offers performance benefits, it comes with risks and should be used judiciously in systems that prioritize consistency over speed.

#### Common Mistakes
& Follow-Ups:

Mistake: Assuming all isolation levels provide the same guarantees regarding data consistency.

#### Follow-Up
Discussing how different transaction management strategies (such as two-phase commit) can mitigate some of these issues, and when to prefer stronger isolation levels over Read Uncommitted.

**Comparison:**

```
Isolation Level	Description
Read Uncommitted	Allows reading uncommitted transactions.
Read Committed	Ensures only committed data is read.
Repeatable Read	Ensures multiple reads within a transaction return the same result.
Serializable	Ensures complete isolation, as if there were no other transactions running.
```

**Flow:**

1. Rendering diagram…

