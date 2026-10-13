---
type: "practice"
difficulty: "Basic"
---

# How would you write a query to find all employees hired in the last 90 days relative to today?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Retention Queries](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Retention_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write an SQL query that finds all employees hired within the last 90 days, we need to understand two key concepts: the use of date functions and simple filtering based on a condition. The core idea is to calculate the difference between the current date and the hire date of each employee, ensuring this difference is less than or equal to 90 days.

The interviewer will be probing your understanding of SQL date handling functions and basic query structure. We assume that:

There is a table employees with at least one column hire_date.
The current date can be obtained using an appropriate function in the database system being used (e.g., CURRENT_DATE() for MySQL, PostgreSQL, or similar).

Here’s how we can break down and solve this problem step-by-step:

Extract Today's Date: Use a built-in SQL function to get today's date.
Calculate the Difference: Subtract the hire date from today's date to find out how many days ago an employee was hired.
Filter Employees: Only select employees who were hired within 90 days.

Now, let’s write this query in SQL:

SQL
SELECT *
FROM employees
WHERE hire_date >= CURRENT_DATE - INTERVAL '90' DAY;

This query selects all columns from the employees table where the hire_date is greater than or equal to today's date minus 90 days. The specific syntax for INTERVAL '90' DAY may vary slightly depending on your database system (e.g., INTERVAL '90 days' in PostgreSQL).

**Code:**

```sql
SELECT *
FROM employees
WHERE hire_date >= CURRENT_DATE - INTERVAL '90' DAY;
```

