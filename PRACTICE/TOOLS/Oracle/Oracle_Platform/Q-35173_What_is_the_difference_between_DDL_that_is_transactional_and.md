---
type: "practice"
difficulty: "Deep"
---

# What is the difference between DDL that is transactional and DDL that implicitly commits in Oracle and why does that matter for migrations?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Oracle Isolation and Read Consistency](<../../../../TOOLS/Oracle/Oracle_Platform/Oracle_Isolation_and_Read_Consistency.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

DDL (Data Definition Language) commands in Oracle can be either transactionally committed or implicitly committed, which significantly impacts database management during migrations and other operations.

#### Triaging the Question

The core of this question revolves around understanding the behavior of DDL statements in different contexts within an Oracle database. Specifically, it seeks to differentiate between DDL commands that are transactional (i.e., committed as part of a larger transaction) and those that implicitly commit (i.e., they take effect immediately without needing a separate COMMIT command). The significance lies in how these differences affect the behavior during migrations or other critical operations.

#### Assumptions
We are working with Oracle Database version 12c or later, where this distinction is most relevant.
Migrations involve significant changes to the database schema and data integrity needs to be maintained throughout the process.

#### First-Principles Solution
Transactional DDL (Implicit Commit):
Behavior: These commands are committed as part of a transaction. For example, when using BEGIN TRANSACTION; or within an autonomous transaction.

#### Example
SQL
-- Example of transactional DDL in Oracle
BEGIN TRANSACTION;
ALTER TABLE employees MODIFY salary NUMBER(10);
COMMIT;  -- Not necessary if the entire block is committed at once
Implicitly Committed DDL (No Explicit Transaction Block):
Behavior: Commands are executed and committed immediately, without needing a separate COMMIT.

#### Example
SQL
-- Example of implicitly committed DDL in Oracle
ALTER TABLE employees MODIFY salary NUMBER(10);
Why This Matters for Migrations:

During migrations, the consistency and integrity of data are paramount. Implicitly committing DDL can lead to unexpected changes if not managed carefully. For instance:

Mistakes: If a migration script includes implicitly committed DDL commands that modify critical tables, they might take effect immediately, potentially disrupting ongoing transactions or violating constraints.

#### Cons
istency: Transactional DDL allows for more control over the transaction boundaries, enabling the rollback of changes if something goes wrong during the migration.
Best Practices:
Use Explicit Transactions: For complex migrations, using explicit transactions ensures that all schema modifications are committed only when desired.
Dry Runs: Perform dry runs of migration scripts to identify implicitly committed DDL statements and adjust them accordingly.
Rollback Plans: Have robust rollback plans in place to revert any changes if something fails.

#### Closing Summary

Understanding the behavior of transactional versus implicitly committed DDL is crucial for database developers, especially during migrations. Explicitly managing transactions can help maintain data integrity and prevent unintended changes that could disrupt ongoing operations.

#### Common Mistakes/Follow-Ups
Ignoring Transaction Boundaries: Developers might inadvertently include implicitly committed DDL in critical scripts without realizing the immediate effect.
Over-Reliance on Implicit Commit: Failing to recognize when explicit transactions are needed can lead to unexpected behavior during migrations or other complex operations.

