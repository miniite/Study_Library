---
type: "practice"
difficulty: "Basic"
---

# How would you find the highest and lowest salary in the employees table using aggregate functions?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Aggregate Precision and Types](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Aggregate_Precision_and_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to identify the highest (maximum) and lowest (minimum) salaries from the employees table. The core idea is to use SQL aggregate functions MAX() and MIN(), which are perfect for finding these values.

#### Triage

The question asks us to find the highest and lowest salary in the employees table. This means we need to understand how to use aggregate functions correctly within an SQL query.

#### Solution

First, let's break down the steps:

Identify the Table and Columns: The relevant column here is salary from the employees table.
Use Aggregate Functions: We'll use MAX() to find the highest salary and MIN() to find the lowest salary.

Here’s how we can write the SQL query:

SQL
SELECT
    MAX(salary) AS highest_salary,
    MIN(salary) AS lowest_salary
FROM
    employees;

#### Closing Summary
+ Common Mistakes/Follow-Ups:

The above query will return two values: highest_salary and lowest_salary. However, it's important to note that if the table is empty or all salaries are null, both MAX() and MIN() could result in null. We should handle such cases appropriately.

#### Common mistakes
include:

Forgetting to use AS for aliasing.
Using non-existent column names.
Not handling scenarios with null values properly.

If you want to ensure that the salary is not null before calculating, you can add a condition using WHERE salary IS NOT NULL.

#### Example
Dry-Run

Assume the employees table has the following sample data:

PLAINTEXT
| id | name    | salary |
|----|---------|--------|
| 1  | Alice   | 50000  |
| 2  | Bob     | 60000  |
| 3  | Carol   | 45000  |
| 4  | David   | NULL   |

The query would return:

PLAINTEXT
| highest_salary | lowest_salary |
|----------------|---------------|
| 60000          | 45000         |
Self-Check

The provided solution is correct and complete. The SQL query uses aggregate functions appropriately, handles the scenario of null values, and provides a clear output.

**Comparison:**

| id | name    | salary |
|----|---------|--------|
| 1  | Alice   | 50000  |
| 2  | Bob     | 60000  |
| 3  | Carol   | 45000  |
| 4  | David   | NULL   |

**Code:**

```sql
SELECT
    MAX(salary) AS highest_salary,
    MIN(salary) AS lowest_salary
FROM
    employees;
```

**Example input / output:**

```text
Input:
SELECT
    MAX(salary) AS highest_salary,
    MIN(salary) AS lowest_salary
FROM
    employees;

Output:
highest_salary | lowest_salary
----------------|---------------
60000          | 45000         |
```

**Data:**

| id | name    | salary |
|----|---------|--------|
| 1  | Alice   | 50000  |
| 2  | Bob     | 60000  |
| 3  | Carol   | 45000  |
| 4  | David   | NULL   |

**Flow:**

1. Start
2. Is table empty or all null?
3. Handle null values
4. Return MAX(salary) ashighest_salary, MIN(salary) aslowest_salary
5. Return NULL for both
6. End Query

