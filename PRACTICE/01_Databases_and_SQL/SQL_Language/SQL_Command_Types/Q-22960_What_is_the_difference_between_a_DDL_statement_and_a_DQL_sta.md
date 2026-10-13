---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a DDL statement and a DQL statement, and where does SELECT fit?

**Difficulty:** Basic · **Levels:** Fresher · [↑ DML Statements](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DML_Statements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database systems, Data Definition Language (DDL), Data Manipulation Language (DML), Data Control Language (DCL), and Transaction Control Language (TCL) are used to manage data in different ways. A DDL statement is used to define the structure of a database, such as creating or altering tables. On the other hand, a DQL statement, which stands for Data Query Language, is primarily used to retrieve data from a database, with "SELECT" being its most common form.

In this answer, we will explore the differences between DDL and DQL statements, focusing on SELECT, and discuss where each type of statement fits in database management.

**Code:**

```sql
-- Example of a DDL statement to create a table
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

-- Example of a DQL (SELECT) statement to retrieve data
SELECT * FROM Employees;
```

**Example input / output:**

```text
-- Input for creating an Employees table
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
);

-- Output after executing the DDL statement
+-----------------------------+
| Operation completed successfully.  |
+-----------------------------+

-- Input for selecting all employees from the Employees table
SELECT * FROM Employees;
```

