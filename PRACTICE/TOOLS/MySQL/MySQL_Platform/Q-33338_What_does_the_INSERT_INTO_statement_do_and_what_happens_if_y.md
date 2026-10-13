---
type: "practice"
difficulty: "Basic"
---

# What does the INSERT INTO statement do and what happens if you omit a column that has no default?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ MySQL Query Basics](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The INSERT INTO statement is used to add new rows of data into an existing table in MySQL. If you attempt to insert values without specifying all columns and one or more of those columns do not have a default value, the operation will fail due to a constraint violation.

#### Triaging the Question

What the question is probing: The core concept here revolves around understanding how INSERT INTO works with respect to table schemas, constraints, and defaults in MySQL. Specifically, it tests your knowledge of what happens when you omit columns that do not have default values.

#### Assumptions

You are familiar with basic SQL syntax.
You understand the concept of tables and columns within a database.
You know about primary keys and non-null constraints.

#### First Principles Solution

Understanding INSERT INTO: The INSERT INTO statement is used to add rows to a table. Its general form is:

SQL
INSERT INTO table_name (column1, column2, column3, ...)
    VALUES (value1, value2, value3, ...);

You can specify the columns explicitly or omit them and rely on position.

Handling Omissions: If you omit a column in the INSERT INTO statement, MySQL will use the default value for that column if one is set. However, if a column does not have a default value (and may also be declared as NOT NULL), this omission leads to an error.

#### Example Scenario
Consider a table employees with columns id, name, and salary. The id column is the primary key and has no default value, while salary does not have a default value.

SQL
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10, 2)
);

Inserting Without Specifying Salary: If you try to insert an employee without specifying the salary column:

SQL
INSERT INTO employees (id, name) VALUES (1, 'John Doe');

This will fail with a constraint violation error because the salary column is NOT NULL and has no default value.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Summary: The INSERT INTO statement in MySQL requires all columns to be specified unless they have default values or are marked as NOT NULL but have a default. Omitting such columns will result in an error.

#### Common Mistakes

Not understanding the importance of constraints like NOT NULL.
Overlooking the necessity of providing default values for columns that do not allow nulls.

#### Follow-Ups

How would you handle cases where multiple columns are missing and have no defaults?
Can you discuss the implications of having a database schema with many non-default NOT NULL columns?

**Comparison:**

| Column Name | Data Type   | Constraints                |
|-------------|------------|---------------------------|
| id          | INT        | PRIMARY KEY, NOT NULL      |
| name        | VARCHAR(100)|                      |
| salary      | DECIMAL(10, 2) | NOT NULL                  |

**Code:**

```sql
-- Example table creation
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10, 2)
);

-- Attempting to insert without specifying all columns
INSERT INTO employees (id, name) VALUES (1, 'John Doe'); -- This will fail

-- Insert with all required values
INSERT INTO employees (id, name, salary) VALUES (1, 'John Doe', 5000.00); -- This works
```

**Example input / output:**

```

Input:

SQL
Copy
INSERT INTO employees (id, name) VALUES (1, 'John Doe');

Output:

PLAINTEXT
Copy
ERROR 1364 (HY000): Field 'salary' doesn't have a default value
```

**Data:**

```sql
-- Table creation schema
CREATE TABLE employees (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    salary DECIMAL(10, 2)
);
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the column with no defaultNOT NULL?
5. Error: Field 'column_name'doesn't have a default value
6. Insert operation succeeds,using default value if present
7. End: Error Handling
8. End: Insertion Successful

