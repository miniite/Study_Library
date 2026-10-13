---
type: "practice"
difficulty: "Basic"
---

# What does the INSERT statement do and what is its basic syntax?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Upserts](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The INSERT statement in SQL is used to add new rows of data into a database table. Its basic syntax includes specifying the table name, followed by values corresponding to each column, and optionally indicating which columns are being updated.

When you're working with a relational database management system (RDBMS), understanding how INSERT works can be crucial for managing your data effectively. This statement is fundamental in any SQL-based project, especially when starting out as a database developer.

#### Triage
The question asks about the functionality and basic syntax of the INSERT statement. It's probing whether you understand one of the most basic operations available in SQL databases. Key assumptions are that you're familiar with tables, columns, and values within an RDBMS context. We'll walk through from first principles to ensure a thorough understanding.

First Principles: In a database table, each row represents data for a particular entity, while columns define the attributes or fields of this entity. To add new data, you use INSERT. The statement specifies the target table and the values corresponding to these columns.

Basic Syntax:

Identify the target table.
Use the INSERT INTO clause followed by the table name.
Specify the column names in parentheses (optional if inserting all columns).
Provide values that match the specified columns or all columns if not specified.
Optionally, you can use a subquery to insert data based on another query.

#### Example

SQL
INSERT INTO employees (id, name, department, salary)
VALUES (101, 'John Doe', 'HR', 65000);

or simply:

SQL
INSERT INTO employees
VALUES (102, 'Jane Smith', 'IT', 80000);

#### Common Mistakes
and Follow-Ups:

Mistake: Forgetting to specify column names when inserting into a table with many columns.

#### Follow-Up
Discussing the implications of not specifying columns if all are being updated, such as potential SQL injection risks.

In practice, INSERT is often used in conjunction with other DDL (Data Definition Language) and TCL (Transaction Control Language) commands to manage data efficiently. Understanding these basics will form a solid foundation for more complex database operations.

**Code:**

```sql
-- Example of inserting into a table with specific columns specified
INSERT INTO employees (id, name, department, salary)
VALUES (101, 'John Doe', 'HR', 65000);

-- Example without specifying columns
INSERT INTO employees
VALUES (102, 'Jane Smith', 'IT', 80000);
```

