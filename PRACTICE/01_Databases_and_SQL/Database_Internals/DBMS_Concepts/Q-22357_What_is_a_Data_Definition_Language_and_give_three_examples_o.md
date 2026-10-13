---
type: "practice"
difficulty: "Basic"
---

# What is a Data Definition Language and give three examples of DDL statements?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Views and Materialized Views](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Views_and_Materialized_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Data Definition Language (DDL) is part of SQL that defines the structure of databases, including creating, altering, and removing database objects such as tables, indexes, or views. Essentially, it helps in organizing data by defining its schema.

In a basic concept interview for a Database Developer fresher, we need to focus on understanding what DDL is and providing examples of common statements used in SQL.

#### Triage

The question asks us to explain DDL and provide three examples of DDL statements. The interviewer wants you to demonstrate your foundational knowledge about database management systems (DBMS) and SQL commands that are essential for creating and modifying database structures.

#### First-Principles Solution
Explanation: Data Definition Language is a subset of SQL used to create, modify, and delete database objects like tables, views, or indexes.

#### Example
s:
CREATE TABLE: Used to define the structure of a new table.
ALTER TABLE: Used to add, change, or remove columns in an existing table.
DROP TABLE: Used to delete a table from the database.

#### Closing Summary
and Common Mistakes/Follow-Ups:

To wrap up, it's important to clearly articulate that DDL is used for defining the structure of databases. Three common examples include:

CREATE TABLE: This statement defines columns, their data types, constraints, etc.
ALTER TABLE: Useful for modifying existing tables by adding or dropping columns, altering column properties, or renaming tables.
DROP TABLE: A destructive command that removes a table from the database.

#### Common mistakes
to avoid include confusing DDL with Data Manipulation Language (DML), which is used for inserting, updating, and deleting data. Always ensure you understand the distinction between these languages.

Common Follow-Ups:
The interviewer might ask about specific constraints or datatypes commonly used in CREATE TABLE.
They could also inquire about when to use ALTER TABLE versus creating a new table.
Another follow-up could be about the implications of using DROP TABLE, such as data loss, backup practices, and recovery strategies.

**Comparison:**

| DDL Statement | Purpose |
|---------------|---------|
| CREATE TABLE  | Defines the structure of a new table. |
| ALTER TABLE   | Modifies an existing table. |
| DROP TABLE    | Deletes a table from the database. |

**Code:**

```sql
-- Example of DDL Statements

-- CREATE TABLE example
CREATE TABLE employees (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    salary DECIMAL(10, 2),
    hire_date DATE
);

-- ALTER TABLE example
ALTER TABLE employees ADD COLUMN department_id INT;

-- DROP TABLE example
DROP TABLE employees;
```

**Example input / output:**

```

-- Input (DDL Statements) CREATE TABLE employees ( id INT PRIMARY KEY, first_name VARCHAR(50), last_name VARCHAR(50), salary DECIMAL(10, 2), hire_date DATE );

ALTER TABLE employees ADD COLUMN department_id INT;

DROP TABLE employees;

CODE
Copy

```sql
-- Output (Result)
Table 'employees' created.
Column 'department_id' added to table 'employees'.
Table 'employees' dropped.
```

**Data:**

```json
{
  "employees": [
    {
      "id": 1,
      "first_name": "John",
      "last_name": "Doe",
      "salary": 5000.00,
      "hire_date": "2023-01-01"
    },
    {
      "id": 2,
      "first_name": "Jane",
      "last_name": "Smith",
      "salary": 6000.00,
      "hire_date": "2023-02-01"
    }
  ]
}
```

**Flow:**

1. No
2. Yes
3. New constraints needed
4. Constraints not needed
5. No
6. Yes
7. Dependencies exist
8. No dependencies
9. CREATE TABLE
10. Does the table already exist?
11. Create new table
12. Check for existing constraints
13. Add constraints to existingtable
14. Modify existing table
15. ALTER TABLE
16. Do you need to add a column?
17. Change or drop columns
18. Add new column(s)
19. DROP TABLE
20. Are there any dependencies?
21. Handle dependencies first
22. Drop table permanently

