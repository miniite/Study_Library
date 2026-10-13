---
type: "practice"
difficulty: "Basic"
---

# How do you add a new column to an existing table and set a default value for it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Safe Migrations on Rolling Deploys](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Safe_Migrations_on_Rolling_Deploys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To add a new column with a default value in PostgreSQL, we need to use SQL commands to modify our database schema. This process involves understanding how to write DDL (Data Definition Language) statements and ensuring data integrity when making changes.

The core idea is straightforward but requires careful execution. We will:

Identify the existing table.
Determine where the new column should be placed.
Define a default value for the new column if needed.
Use the ALTER TABLE command to add the column with its default value.

Let's break this down step by step from first principles:

#### Triage

The question is asking how to modify an existing table in PostgreSQL by adding a new column and setting a default value for it. The key point here is using the appropriate SQL syntax, which involves understanding DDL commands specifically tailored for adding columns.

#### Assumptions
We have access to the database and can execute SQL commands.
The table we are modifying already exists with some data in it (if applicable).

#### First-Principles Solution

To add a new column with a default value, we use the ALTER TABLE statement. The syntax for this is:

SQL
ALTER TABLE table_name
ADD COLUMN column_name data_type [DEFAULT default_value];

For example, if you have a table named employees and want to add an age column with a default value of 30, the SQL command would look like this:

SQL
ALTER TABLE employees
ADD COLUMN age INT DEFAULT 30;

#### Closing Summary
+ Common Mistakes/Follow-Ups

Adding a new column is a straightforward task in PostgreSQL, but there are some common pitfalls to avoid. First, always ensure you have a backup of your database before making schema changes. Additionally, if the table has existing data and the column type requires values (like INT), you might need to handle NULLs or other default values.

Here’s a worked example:

Worked Example

Assume we have an employees table with the following structure:

SQL
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    hire_date DATE
);

We want to add an age column with a default value of 30. The SQL command would be:

SQL
ALTER TABLE employees
ADD COLUMN age INT DEFAULT 30;

After running this command, the employees table schema will look like this:

Column Name	Data Type
id	SERIAL
name	VARCHAR
hire_date	DATE
age	INT

#### Common Mistakes
Forgetting to backup: Always back up your database before making changes.
Incorrect data type: Ensure the default value is appropriate for the column's data type.
Handling existing NULLs: If the table already has rows, adding a non-nullable column might require additional steps (like setting default values or allowing NULLs).
Recap

Adding a new column with a default value in PostgreSQL involves using the ALTER TABLE command followed by ADD COLUMN. Ensure you understand the data type and default value constraints. Always back up your database before making schema changes.

