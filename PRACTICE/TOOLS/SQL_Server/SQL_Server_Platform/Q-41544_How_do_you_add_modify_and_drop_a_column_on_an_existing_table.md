---
type: "practice"
difficulty: "Basic"
---

# How do you add, modify, and drop a column on an existing table using ALTER TABLE?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MERGE and Upserts](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/MERGE_and_Upserts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add, modify, or drop a column in an existing table using ALTER TABLE in SQL Server, we need to understand the syntax of these operations and how they impact the structure of our database. Here’s a step-by-step breakdown from first principles.

Firstly, let's consider the basic idea: modifying an existing table structure typically involves adding new columns or making changes without changing the schema too drastically. Dropping columns is more straightforward but can be dangerous if not done carefully.

#### Triaging the Question

The question asks how to use ALTER TABLE for three main operations:

Adding a Column: This operation requires specifying the column name and data type.
Modifying a Column: This might involve changing its data type, adding constraints, or renaming it.
Dropping a Column: This involves removing an existing column from the table.

#### Assumptions
We are working with SQL Server.
The database schema is already designed and contains at least one table.
Basic knowledge of DDL (Data Definition Language) commands.

#### First-Principles Solution

Adding a Column:

SQL
ALTER TABLE [TableName]
ADD [NewColumnName] [DataType] [ColumnOptions];
TableName: The name of the table where we want to add the column.
NewColumnName: The name for the new column.
DataType: The data type of the new column (e.g., INT, VARCHAR(255)).
ColumnOptions can include additional constraints like NOT NULL or DEFAULT.

#### Example

SQL
ALTER TABLE Employees
ADD Age INT;

Modifying a Column:

To modify a column, we use the MODIFY COLUMN clause if supported by the SQL Server version.
SQL
ALTER TABLE [TableName]
ALTER COLUMN [OldColumnName] [NewDataType];
In earlier versions of SQL Server, you might need to drop and recreate the column.

#### Example

SQL
ALTER TABLE Employees
ALTER COLUMN Age INT NOT NULL;

Dropping a Column:

Dropping a column is straightforward but should be done with caution as it permanently removes the column.
SQL
ALTER TABLE [TableName]
DROP COLUMN [OldColumnName];
Ensure you understand the implications of dropping columns, especially in tables that are already being used.

#### Example

SQL
ALTER TABLE Employees
DROP COLUMN Age;

#### Closing Summary
and Common Mistakes/Follow-Ups
Always back up your database before performing schema modifications.
Verify the impact on any dependent objects such as views, stored procedures, or other tables.
Ensure you understand the constraints (if any) associated with the columns being modified.

#### Common mistakes
include:

Not checking the data type compatibility when modifying a column.
Forgetting to back up the database before making changes.
Accidentally dropping critical columns that are used elsewhere in the application.

#### Example
Walkthrough

#### Cons
ider we have an Employees table and want to add, modify, and drop a column:

Adding Age:

SQL
ALTER TABLE Employees
ADD Age INT;

Modifying Age (Ensuring NOT NULL):

SQL
ALTER TABLE Employees
ALTER COLUMN Age INT NOT NULL;

Dropping Age:

SQL
ALTER TABLE Employees
DROP COLUMN Age;
Final Self-Check

This solution is correct and covers all aspects of the question, ensuring completeness.

**Code:**

```sql
-- Adding a column
ALTER TABLE Employees
ADD Age INT;

-- Modifying a column
ALTER TABLE Employees
ALTER COLUMN Age INT NOT NULL;

-- Dropping a column
ALTER TABLE Employees
DROP COLUMN Age;
```

