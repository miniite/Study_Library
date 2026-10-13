---
type: "practice"
difficulty: "Basic"
---

# What is the difference between HAVING and WHERE when you want to filter groups by an aggregate feature threshold?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ GROUP BY vs PARTITION BY](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clarify, both WHERE and HAVING clauses in SQL are used for filtering data; however, they operate at different stages of query execution and serve distinct purposes when working with aggregated features. The key difference lies in the context where each clause is applied.

#### Triaging the Question
Question: You need to filter groups by an aggregate feature threshold.
What's being probed: Understanding the correct use of WHERE vs. HAVING for filtering data based on grouped aggregates.

#### Assumptions
The goal is to identify and process only those groups that meet a specific threshold condition after aggregation.

#### First Principles Solution
Understanding WHERE: This clause filters rows before any GROUP BY operation takes place. It works directly with the original dataset, applying conditions based on individual columns.
Understanding HAVING: This clause is used to filter groups of records that have been aggregated by a GROUP BY statement. It operates after the GROUP BY has been applied and cannot be used in the same query without it.
Detailed Explanation

WHERE Clause:

Used for filtering rows based on individual column values before any grouping.

#### Example
SQL
SELECT department, COUNT(*) as num_employees
FROM employees
WHERE salary > 50000
GROUP BY department;
This query will first filter all employees with a salary greater than 50000 and then group by department.

HAVING Clause:

Used for filtering groups after the aggregate operation has been performed.

#### Example
SQL
SELECT department, COUNT(*) as num_employees
FROM employees
GROUP BY department
HAVING COUNT(*) > 5;
This query will first group by department and then filter out departments that have fewer than 5 employees.
Practical Example

Let's consider an example where we want to find all departments with more than 10 employees.

SQL
-- Using WHERE clause (incorrect approach for this scenario)
SELECT department, COUNT(*) as num_employees
FROM employees
WHERE department in (
    SELECT department
    FROM employees
    GROUP BY department
    HAVING COUNT(*) > 10
);
Explanation: This query is incorrect because the WHERE clause filters individual rows before grouping, not after.
SQL
-- Using HAVING clause (correct approach)
SELECT department, COUNT(*) as num_employees
FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
Explanation: Here, we group by departments and then use HAVING to filter out groups with fewer than 10 employees.

#### Common Mistakes
and Follow-Ups
Mistake: Using WHERE instead of HAVING for filtering groups.

#### Follow-Up
What happens if you mistakenly put a condition in the wrong clause?
If you use WHERE, it will not filter out groups but rather individual rows, leading to incorrect results.

#### Closing Summary

In summary, WHERE filters individual rows before grouping, while HAVING filters grouped aggregates after they have been calculated. Always ensure that your conditions are placed in the correct clause based on whether you are filtering individual rows or aggregated groups.

**Comparison:**

```
id	name	department	salary
1	Alice	Sales	60000
2	Bob	Sales	70000
3	Charlie	HR	55000
4	David	Engineering	80000
5	Eve	Engineering	90000
Output
WHERE Query: Will not produce any results since no department has more than 10 employees.
HAVING Query: Returns the Engineering department as it has 2 employees.
```

**Code:**

```sql
-- Example of incorrect use of WHERE
SELECT department, COUNT(*) as num_employees
FROM employees
WHERE department in (
    SELECT department
    FROM employees
    GROUP BY department
    HAVING COUNT(*) > 10
);

-- Correct example using HAVING
SELECT department, COUNT(*) as num_employees
FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
```

**Example input / output:**

```text
-- Using WHERE (incorrect)
SELECT department, COUNT(*) as num_employees
FROM employees
WHERE department in (
    SELECT department
    FROM employees
    GROUP BY department
    HAVING COUNT(*) > 10
);

-- Correct example using HAVING
SELECT department, COUNT(*) as num_employees
FROM employees
GROUP BY department
HAVING COUNT(*) > 10;
```

**Data:**

```sql
CREATE TABLE employees (
    id INT,
    name VARCHAR(50),
    department VARCHAR(50),
    salary DECIMAL
);
INSERT INTO employees (id, name, department, salary) VALUES
(1, 'Alice', 'Sales', 60000),
(2, 'Bob', 'Sales', 70000),
(3, 'Charlie', 'HR', 55000),
(4, 'David', 'Engineering', 80000),
(5, 'Eve', 'Engineering', 90000);
```

**Flow:**

1. SELECT
2. GROUP BY
3. HAVING COUNT(*) > 10
4. WHERE salary > 50000

