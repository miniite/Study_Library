---
type: "practice"
difficulty: "Basic"
---

# How do you insert multiple rows into a table with a single INSERT statement?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ RETURNING Clause](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/RETURNING_Clause.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To insert multiple rows into a PostgreSQL table in a single INSERT statement, you can list each row's values directly within the SQL query. This approach is efficient and straightforward. We'll break down the process from first principles to understand how it works.

Firstly, let’s consider what the question is asking: The goal is to insert several rows of data into an existing table with a single INSERT statement. PostgreSQL supports this by allowing you to list multiple sets of values in the VALUES clause, separated by commas.

#### Assumptions

You have already created a table and know its schema (i.e., column names and their types).
The number and type of columns must match the provided data.
Each row of data should be enclosed in parentheses and separated by commas within the VALUES clause.

Step-by-step reasoning from first principles:

Understand the Table Structure: Identify the table name, its columns, and their types.
Prepare the Data Rows: Create a list of rows where each row contains values for all the columns in the same order as they appear in the table schema.

#### Cons
truct the INSERT Statement: Use the VALUES clause to insert multiple rows by separating them with commas.

#### Example
Suppose we have a table named students with the following schema: id INT PRIMARY KEY, name VARCHAR(100), age INT.

SQL
CREATE TABLE students (
    id SERIAL,
    name VARCHAR(100),
    age INT
);

To insert multiple rows into this table, you can use:

SQL
INSERT INTO students (name, age)
VALUES
('Alice', 25),
('Bob', 30),
('Charlie', 28);

This single INSERT statement adds three new records to the students table.

#### Common mistakes
and follow-ups:

Mismatched Column Count: Ensure that each row in the VALUES clause has exactly as many values as there are columns.
Type Mismatch: Make sure that the data types match those of the corresponding columns.
Order Error: The order of values in each row must correspond to the column order.

By following these steps, you can efficiently insert multiple rows into a PostgreSQL table with a single INSERT statement.

