---
type: "practice"
difficulty: "Basic"
---

# How would you update a column value based on values in the same row?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ DML Statements](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Command_Types/DML_Statements.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When we talk about updating a column's value based on other columns within the same row, we are essentially performing an operation that uses conditions or expressions to modify data. This task can be handled using DML (Data Manipulation Language) commands in SQL, particularly with the UPDATE statement. In this context, we'll use a combination of DDL and TCL (Transaction Control Language) if necessary.

To start, let's break down what is involved:

Understanding the Table Schema: Knowing the table structure and the columns involved.
Identifying the Condition: The specific condition under which you want to update a column.
Specifying the New Value: Determining how the new value should be calculated based on existing data.

Let's walk through an example step-by-step:

#### Triage

The question is asking about updating a column in a row based on values from other columns within the same row. This involves using the UPDATE statement with a conditional expression to modify the data according to specific criteria. The main concepts involved are DML and possibly TCL for transaction management.

#### First-Principles Solution

Assume we have a table named employees with the following schema:

SQL
CREATE TABLE employees (
    id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    salary DECIMAL(10, 2)
);

Suppose we want to update the salary column based on a condition involving both first_name and last_name. For instance, if the employee's name contains "Smith" in either the first or last name, we might want to double their salary.

Step-by-Step Solution
Identify the Condition: We need to check if the first_name OR last_name contains "Smith".
Determine the New Value: If the condition is met, set the new value for the salary.

Here’s how we can write this in SQL:

SQL
UPDATE employees
SET salary = salary * 2
WHERE first_name LIKE '%Smith%' OR last_name LIKE '%Smith%';

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to update a column based on values from the same row, we use an UPDATE statement with appropriate conditions. Key points include:

Using Conditions: The WHERE clause ensures that only relevant rows are updated.
Transaction Control (Optional): For critical operations, consider wrapping your UPDATE statements in a transaction using TCL commands like BEGIN, COMMIT, or ROLLBACK.

#### Common mistakes
include forgetting to specify the WHERE condition, leading to unintended data modifications. Always ensure that the update logic is clear and test with sample data before running on production.

For follow-ups, one might ask about handling concurrent updates, dealing with large datasets, or optimizing performance.

**Code:**

```sql
UPDATE employees
SET salary = salary * 2
WHERE first_name LIKE '%Smith%' OR last_name LIKE '%Smith%';
```

