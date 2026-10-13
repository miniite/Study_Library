---
type: "practice"
difficulty: "Basic"
---

# What is the FIRST_VALUE() function and when would you use it?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ ROW_NUMBER RANK and DENSE_RANK](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/ROW_NUMBER_RANK_and_DENSE_RANK.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The FIRST_VALUE() function in SQL is a window function that allows you to retrieve the first value of an expression from the partition of a result set, based on a specified ordering. Essentially, it helps you access the initial row's value within a frame of rows defined by a partition and order.

In essence, the question is asking us to understand how FIRST_VALUE() works and scenarios where this function would be useful in database development. Let’s break down the concept step-by-step from first principles.

**Comparison:**

| employee_id | department_id | first_hire_date  |
| ----------- | ------------- | -----------------|
| 1          | 10            | 2015-03-17       |
| 2          | 10            | 2015-03-17       |
| 3          | 11            | 2014-05-18       |
| 4          | 11            | 2014-05-18       |

**Code:**

```sql
-- Example usage of FIRST_VALUE()
SELECT
    employee_id,
    department_id,
    FIRST_VALUE(hire_date) OVER (PARTITION BY department_id ORDER BY hire_date ASC) AS first_hire_date
FROM
    employees;

This query returns the earliest hire date for each department by partitioning the result set based on department_id and ordering it by hire_date.
```

**Example input / output:**

```sql
-- Dry-run input -> output
Input:
SELECT
    employee_id,
    department_id,
    FIRST_VALUE(hire_date) OVER (PARTITION BY department_id ORDER BY hire_date ASC) AS first_hire_date
FROM
    employees;

Output:
|employee_id|department_id|first_hire_date  |
|-----------|-------------|-----------------|
|1          |10           |2015-03-17       |
|2          |10           |2015-03-17       |
|3          |11           |2014-05-18       |
|4          |11           |2014-05-18       |

This output shows the first hire date for each department, as expected.
```

**Data:**

```json
{
  "employees": [
    {"employee_id": 1, "department_id": 10, "hire_date": "2015-03-17"},
    {"employee_id": 2, "department_id": 10, "hire_date": "2016-04-19"},
    {"employee_id": 3, "department_id": 11, "hire_date": "2014-05-18"},
    {"employee_id": 4, "department_id": 11, "hire_date": "2017-06-20"}
  ]
}

The JSON data represents a table of employees with their respective employee_id, department_id, and hire_date.
```

**Flow:**

1. Yes
2. No
3. SELECT employee_id,department_id
4. Partition by department_id?
5. ORDER BY hire_date ASC
6. FIRST_VALUE(hire_date) withinthe frame
7. Skip partitioning and order
8. Return first_hire_date for eachrow
9. This flowchart illustrates the process of using `FIRST_VALUE()` with a window function.

