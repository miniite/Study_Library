---
type: "practice"
difficulty: "Basic"
---

# What does the ALTER TABLE statement let you do and give two examples of its use?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MySQL Query Basics](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Alright, let's dive into the ALTER TABLE statement in MySQL! It’s a powerful tool that allows us to modify the structure of an existing table. In essence, it lets you add, drop, or change columns, and even alter indexes and constraints.

#### Triaging the Question

The question is asking two things:

What can the ALTER TABLE statement do?
Provide two examples of its use.

#### Assumptions
We are working with a MySQL database.
The context assumes you have some familiarity with basic SQL operations like creating and querying tables.

#### Solution
from First Principles

The ALTER TABLE statement in MySQL is used to change the structure of an existing table. Here’s what it can do:

Add Columns: You can add new columns to a table.
Drop Columns: You can remove columns from a table.
Rename Columns: You can rename existing columns.
Modify Column Types and Attributes: You can change the data type of a column, set or modify constraints (like adding NOT NULL).
Add/Remove Indexes: You can add or drop indexes to optimize query performance.

Now let’s look at some specific examples:

#### Example
1: Adding a New Column

Suppose you have an existing table named employees with the following structure:

SQL
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255)
);

You want to add a new column for storing the employee's hire date. You can use the ALTER TABLE statement like this:

SQL
ALTER TABLE employees ADD COLUMN hire_date DATE;

This command adds a new column called hire_date of type DATE.

#### Example
2: Renaming and Modifying a Column

#### Cons
ider that you have the same employees table, but now you realize that the email field should be named contact_email. Additionally, you want to change its data type from VARCHAR(255) to TEXT because some emails are longer. You can use:

SQL
ALTER TABLE employees CHANGE COLUMN email contact_email TEXT;

This command renames the column email to contact_email and changes its data type.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The ALTER TABLE statement is incredibly versatile, allowing you to dynamically adjust your database schema as needed. However, it’s important to be cautious when using this command, especially in production environments, because making structural changes can have unintended side effects on existing data and queries.

#### Common Mistakes

Not backing up the database before making significant schema changes.
Failing to consider the impact of adding or modifying constraints (e.g., ensuring referential integrity).
Overuse of ALTER TABLE for simple, non-structural changes that could be handled by other SQL commands like UPDATE.

#### Follow-ups
might include discussing specific scenarios where certain operations are more appropriate than others.

**Code:**

```sql
-- Example 1: Adding a new column
ALTER TABLE employees ADD COLUMN hire_date DATE;

-- Example 2: Renaming and modifying a column
ALTER TABLE employees CHANGE COLUMN email contact_email TEXT;
```

**Data:**

```sql
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(255),
    email VARCHAR(255)
);
```

