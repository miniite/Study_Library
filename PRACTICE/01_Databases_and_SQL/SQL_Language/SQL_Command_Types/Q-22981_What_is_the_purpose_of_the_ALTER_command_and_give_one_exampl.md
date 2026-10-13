---
type: "practice"
difficulty: "Basic"
---

# What is the purpose of the ALTER command and give one example use?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DROP and CASCADE](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DROP_and_CASCADE.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The ALTER command in SQL is used to modify or change existing database objects, such as tables, views, or indexes. It allows you to add or remove columns, change column properties, and more without deleting the entire object and recreating it from scratch. For example, let's consider altering a table.

#### Triaging the Question

The question asks about the purpose of the ALTER command and provides an example use case. The key points here are:

What is the ALTER command used for?
Provide an example to illustrate its usage.

#### First-Principles Solution
Purpose of ALTER Command

The primary purpose of the ALTER command is to make changes to a database schema dynamically without affecting existing data. This includes modifying table structures, adding or removing constraints, and altering column properties like data types or sizes.

#### Example
Usage

Let's say we have an employees table with some initial structure:

SQL
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    department_id INT
);

Suppose we want to add a new column for the employee’s hire date. We can use the ALTER command as follows:

SQL
ALTER TABLE employees
ADD COLUMN hire_date DATE;

This command adds a new column named hire_date of type DATE to the employees table, allowing us to store and manage this additional information without altering any existing data.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the ALTER command is crucial for maintaining the flexibility of your database schema. It enables you to evolve your database design over time by making incremental changes rather than starting from scratch. Some common mistakes when using ALTER include:

Failing to back up the database before making significant alterations.
Not considering potential cascading effects on dependent objects like views or indexes.
Overusing ALTER for large-scale structural changes that might be better handled with a combination of backup, schema modifications, and restoration.

Common follow-ups an interviewer might ask include:

What are some other operations you can perform using the ALTER command?
How do you manage data integrity during such alterations?

**Code:**

```sql
-- Example of adding a column to an existing table
ALTER TABLE employees
ADD COLUMN hire_date DATE;
```

