---
type: "practice"
difficulty: "Basic"
---

# What is the difference between the AVG, SUM, MIN, and MAX aggregate functions when NULLs are present?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MySQL Query Basics](<../../../../TOOLS/MySQL/MySQL_Platform/MySQL_Query_Basics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with MySQL, understanding how AVG, SUM, MIN, and MAX handle NULL values is crucial for accurate data analysis. In this overview, we'll explore each function's behavior when encountering NULLs, step by step from first principles.

Let’s start with a brief summary:

NULLs in SQL: When dealing with SQL aggregate functions like AVG, SUM, MIN, and MAX, it's important to understand how they treat NULL values. In MySQL, these functions can behave differently depending on whether the input contains NULLs.

Now let’s break down what each function does when a NULL is present:

SUM: The SUM function ignores NULL values.
MIN: The MIN function also ignores NULL values.
MAX: Similarly, the MAX function will ignore NULL values.
AVG: This is where things get interesting. In MySQL, AVG behaves as if NULLs are 0 for both numerator and denominator.

To illustrate this with an example, consider a table employees with columns id, salary, and is_active. Let's assume the following data:

id	salary	is_active
1	5000	1
2	NULL	1
3	7000	0

To find the average active employee salary, we would use:

SQL
SELECT AVG(salary) FROM employees WHERE is_active = 1;

In this case:

SUM(salary) would be 5000 + NULL, resulting in 5000 (since NULLs are ignored).
The count of active employees (COUNT(salary) where is_active = 1) would be 2.

Thus, the AVG function calculates (5000 / 2) = 2500.

Now, let’s look at a step-by-step solution using these principles:

Step-by-Step Solution
Identify Aggregate Function: Decide which aggregate function you need to use (e.g., SUM, AVG).
Filter NULLs: Ensure the query filters out rows with NULL values in relevant columns.
Handle NULLs Explicitly:
SUM and its variations ignore NULLs.
MIN and MAX similarly ignore NULLs.

For instance, to find the average salary of active employees:

SQL
SELECT AVG(salary) FROM employees WHERE is_active = 1;

This query works by ignoring any rows where salary is NULL when calculating the average. It effectively treats NULL as if it’s a zero value in the calculation.

#### Closing Summary

In summary, SUM, MIN, and MAX ignore NULL values in their calculations, while AVG includes them but considers NULLs as zeros for both the numerator and denominator. Understanding this behavior is key to writing accurate SQL queries that handle NULL values appropriately.

#### Common Mistakes

Forgetting that AVG treats NULLs as 0.
Misinterpreting the results of aggregate functions when NULLs are involved.

#### Follow-Ups

How do you handle cases where all values in a column might be NULL?
What about other aggregate functions like COUNT or GROUP BY?

**Code:**

```sql
-- Example SQL to demonstrate AVG behavior with NULLs
SELECT AVG(salary) FROM employees WHERE is_active = 1;
```

**Example input / output:**

```text
-- Input:
SELECT AVG(salary) FROM employees WHERE is_active = 1;

-- Output:
2500
```

**Data:**

| id | salary | is_active |
|----|--------|-----------|
| 1  | 5000   | 1         |
| 2  | NULL   | 1         |
| 3  | 7000   | 0         |

