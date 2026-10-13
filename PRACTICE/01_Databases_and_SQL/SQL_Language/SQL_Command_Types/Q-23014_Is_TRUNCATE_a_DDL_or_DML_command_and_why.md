---
type: "practice"
difficulty: "Intermediate"
---

# Is TRUNCATE a DDL or DML command and why?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML Statements](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DML_Statements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TRUNCATE is an interesting case in the world of SQL commands, often discussed in the context of Data Definition Language (DDL), Data Manipulation Language (DML), Data Control Language (DCL), and Transaction Control Language (TCL). In this answer, we will explore why TRUNCATE is considered a DDL command.

To triage the question, let's first understand what these acronyms mean:

DDL: This stands for Data Definition Language. It includes commands to define or modify database objects such as tables, indexes, and constraints.
DML: This stands for Data Manipulation Language. It includes commands that manipulate data in a table, like INSERT, UPDATE, and DELETE.
DCL: This stands for Data Control Language. It includes commands for managing user permissions and access rights to the database objects.
TCL: This stands for Transaction Control Language. It involves commands related to transaction management, such as committing or rolling back transactions.

Now, let's delve into why TRUNCATE is classified as a DDL command from first principles:

Purpose of TRUNCATE:

TRUNCATE is used to remove all rows from a table but retains the structure and constraints of that table.
It is faster than a combination of DELETE followed by COMMIT, especially when dealing with large tables, as it does not log each row deletion.

Comparison with DML Commands:

While TRUNCATE can be used to remove all rows from a table (similar to using DELETE without any conditions), its primary purpose is more closely aligned with the definition and management of database objects.
TRUNCATE does not allow for the specification of criteria, such as deleting only certain rows based on some condition. This makes it less like DML commands.

Behavior Differences:

DELETE: Logs each row deletion, which can impact the transaction log and performance, especially with large datasets.
TRUNCATE: Does not log individual deletions but instead logs a single operation that deallocates the table's space, making it much faster for bulk operations.

#### Cons
traints and Integrity:

After executing TRUNCATE, all indexes associated with the table are re-built from scratch, ensuring consistency.
This is more akin to defining or altering a database object rather than manipulating data within it.

Commit Behavior:

Unlike other DML commands, TRUNCATE does not allow for rollback once the operation is committed.
It requires a commit (or an autocommit) to finalize the action, aligning more closely with the lifecycle of creating or altering database objects.

Usage Examples:

SQL
-- Example: Truncating a table
TRUNCATE TABLE employees;

#### Closing Summary
In conclusion, TRUNCATE is classified as a DDL command because it fundamentally changes the state of a table by removing all its rows while maintaining the table's structure and constraints. This operation is more aligned with the definition and management of database objects rather than the manipulation of individual data records.

#### Common Mistakes/Follow-Ups

Mistake: Confusing TRUNCATE with DELETE in scenarios where only data needs to be removed, leading to unnecessary transaction logs.

#### Follow-Up
Understanding when it is appropriate to use TRUNCATE versus DELETE, especially in terms of performance and data integrity considerations.

**Code:**

```sql
-- Example usage of TRUNCATE
TRUNCATE TABLE employees;
```

