---
type: "practice"
difficulty: "Basic"
---

# How do you rename a column and a table in SQL?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML Statements](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DML_Statements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To rename a column or a table in SQL, we use Data Definition Language (DDL) commands to define the structure of databases and their objects. Specifically, renaming involves using the ALTER TABLE statement for columns and tables, which is part of Transaction Control Language (TCL). In this response, I'll cover both processes step-by-step.

Firstly, let's understand what DDL/DML/DCL/TCL mean:

DDL: Data Definition Language includes commands like CREATE, DROP, and ALTER to define the structure of databases.
DML: Data Manipulation Language involves operations on data in a database using commands like SELECT, INSERT, UPDATE, and DELETE.
DCL: Data Control Language deals with granting access rights to users, such as GRANT and REVOKE.
TCL: Transaction Control Language includes commands that manage transactions, ensuring consistency and integrity. It typically involves commands like COMMIT, ROLLBACK, and SAVEPOINT.

Now, let's dive into the specific questions:

Renaming a Column:

We use the ALTER TABLE statement with the RENAME COLUMN clause.
The basic syntax is: ALTER TABLE table_name RENAME COLUMN old_column_name TO new_column_name;

Renaming a Table:

Again, we use the ALTER TABLE statement but in this case without specifying any column changes.
The syntax is: ALTER TABLE old_table_name RENAME TO new_table_name;

Let's walk through an example to understand these processes better.

