---
type: "practice"
difficulty: "Basic"
---

# What are DDL, DML, DCL, and TCL categories of SQL commands in Oracle?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ PL/SQL Overview](<../../../../TOOLS/Oracle/Oracle_Platform/PLSQL_Overview.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To get a clear understanding of how to manage data in an Oracle database, it’s essential to familiarize yourself with the different types of SQL commands that fall under DDL (Data Definition Language), DML (Data Manipulation Language), DCL (Data Control Language), and TCL (Transaction Control Language). Let's dive into each category step by step.

#### Triaging the Question

The question asks you to explain the categories of SQL commands in Oracle, specifically DDL, DML, DCL, and TCL. The interviewer is probing your knowledge of these fundamental concepts that form the backbone of database management. Here are some assumptions:

You know what SQL (Structured Query Language) is.
You understand the basic functions of databases like creating tables, inserting data, and controlling access.

#### First Principles Solution
DDL (Data Definition Language)

DDL commands are used to define or alter a schema in the database. These commands do not modify any existing data; instead, they change the structure of the database. Key commands include:

CREATE: Used to create objects like tables, indexes, and views.
SQL
CREATE TABLE employees (
    id NUMBER PRIMARY KEY,
    name VARCHAR2(100),
    age NUMBER
);
ALTER: Modifies an existing schema object.
SQL
ALTER TABLE employees ADD (email VARCHAR2(100));
DROP: Removes a schema object from the database.
SQL
DROP TABLE employees;
DML (Data Manipulation Language)

DML commands are used to manipulate or modify existing data in a table. These commands do not affect the structure of the database but only its content. Key commands include:

SELECT: Retrieves data from one or more tables.
SQL
SELECT * FROM employees;
INSERT: Adds new rows to a table.
SQL
INSERT INTO employees (id, name, age) VALUES (1, 'John Doe', 30);
UPDATE: Modifies existing data in a table.
SQL
UPDATE employees SET age = 31 WHERE id = 1;
DELETE: Removes rows from a table.
SQL
DELETE FROM employees WHERE id = 1;
DCL (Data Control Language)

DCL commands are used to control access permissions and privileges on the data in the database. These commands affect who can do what with specific objects or tables. Key commands include:

GRANT: Allows users or roles to perform certain actions.
SQL
GRANT SELECT, INSERT ON employees TO user1;
REVOKE: Removes previously granted privileges.
SQL
REVOKE SELECT, INSERT ON employees FROM user1;
TCL (Transaction Control Language)

TCL commands are used to manage the transactional integrity of operations. They ensure that a series of SQL statements are treated as a single unit of work. Key commands include:

COMMIT: Commits all changes made within a transaction.
SQL
COMMIT;
ROLLBACK: Reverts any uncommitted changes back to their original state.
SQL
ROLLBACK;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, DDL commands are for defining the structure of a database; DML commands manipulate data within tables; DCL commands control access permissions; and TCL commands manage transactions. Understanding these categories is crucial for effective database management.

#### Common mistakes
include confusing the functions of each category or not fully understanding how transactions work. For follow-ups, you might be asked to provide specific examples or discuss best practices related to each command type.

