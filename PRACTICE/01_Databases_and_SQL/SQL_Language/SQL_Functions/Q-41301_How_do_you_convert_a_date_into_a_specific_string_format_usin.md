---
type: "practice"
difficulty: "Intermediate"
---

# How do you convert a date into a specific string format using functions like TO_CHAR or FORMAT?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Date Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Date_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To convert a date into a specific string format in SQL, we can use functions like TO_CHAR (Oracle) or FORMAT (SQL Server). These functions allow us to specify various formatting options to get our desired output. Here’s a step-by-step guide on how to achieve this.

#### Triaging the Question

The question is essentially asking how to format dates in SQL databases using specific built-in functions. The interviewer might be probing your understanding of date handling and string manipulation within SQL. We'll assume you are familiar with basic SQL operations and are comfortable working with date columns.

#### First-Principles Solution

Let's consider an example where we have a birth_date column in the employees table, and we want to format it into "DD-Mon-YYYY" for Oracle or "dd MMM yyyy" for SQL Server. Here’s how you can do it step-by-step.

Step 1: Understanding the Date Format

For Oracle:

SQL
TO_CHAR(birth_date, 'DD-MON-YYYY')

For SQL Server:

SQL
FORMAT(birth_date, 'dd MMM yyyy', 'en-US')
Step 2: Applying the Function to a Query

Let's create an example query to demonstrate this:

SQL
-- Oracle Example
SELECT employee_id, TO_CHAR(birth_date, 'DD-Mon-YYYY') AS formatted_birth_date
FROM employees;

-- SQL Server Example
SELECT employee_id, FORMAT(birth_date, 'dd MMM yyyy', 'en-US') AS formatted_birth_date
FROM employees;

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, to format dates in SQL, you can use functions like TO_CHAR (Oracle) or FORMAT (SQL Server). Ensure that the date column exists in your table and is correctly typed. Always check for any potential issues with null values, as these might require additional handling.

#### Common mistakes
include:

Incorrect Date Format: Using a format string that doesn’t match what you need.
Handling Null Values: Forgetting to account for null dates, which can cause errors in the query.
Additional Follow-Ups
How would you handle null values in birth_date?
Can you write a function or stored procedure to dynamically generate the formatted date based on user input?

**Code:**

```sql
-- Oracle Example
SELECT employee_id, TO_CHAR(birth_date, 'DD-Mon-YYYY') AS formatted_birth_date
FROM employees;

-- SQL Server Example
SELECT employee_id, FORMAT(birth_date, 'dd MMM yyyy', 'en-US') AS formatted_birth_date
FROM employees;
```

