---
type: "practice"
difficulty: "Intermediate"
---

# How do you find the second highest salary in each department using a window function?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Top-N per Group](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To find the second highest salary in each department, we can leverage SQL's window functions to rank salaries within each department and then select the appropriate row. The core idea here is to use ROW_NUMBER() or similar ranking function to assign a unique number to each salary based on its value within the department. We'll then filter for the second-highest ranked salary.

#### Triage

The question asks us to find the second highest salary in each department using window functions. This means we need to understand how to rank rows, filter them, and ensure that our query is efficient enough for larger datasets. The main assumptions are:

Each row has a salary and a department_id.
There may be duplicate salaries within a department.

#### First-Principles Solution
Rank Salaries: Use the ROW_NUMBER() function to assign a unique number to each salary based on its value within the department.
Filter for Second-Highest Salary: Select only those rows where the rank is 2.

Here’s how we can achieve this in SQL:

SQL
WITH RankedSalaries AS (
    SELECT
        department_id,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) as rn
    FROM
        employees
)
SELECT
    department_id,
    salary
FROM
    RankedSalaries
WHERE
    rn = 2;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query correctly finds the second highest salary in each department. It uses a common table expression (CTE) to rank salaries within departments first and then filters out the desired row.

#### Common Mistakes

Forgetting to partition by department_id when ranking.
Not handling cases where there are fewer than 2 unique salaries per department, which would result in no second highest salary.
Using incorrect window functions (e.g., RANK() or DENSE_RANK()) instead of ROW_NUMBER().

#### Follow-Ups

Discussing the performance implications if the dataset is very large.

#### Cons
idering edge cases like departments with only one employee, or departments where all employees have identical salaries.

**Comparison:**

| department_id | salary |
|---------------|--------|
| 10            | 4800   |
| 20            | 5900   |

**Code:**

```sql
WITH RankedSalaries AS (
    SELECT
        department_id,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department_id ORDER BY salary DESC) as rn
    FROM
        employees
)
SELECT
    department_id,
    salary
FROM
    RankedSalaries
WHERE
    rn = 2;
```

**Example input / output:**

```

Given the sample data:

JSON
Copy
[
    {"id": 1, "name": "Alice", "department_id": 10, "salary": 5000},
    {"id": 2, "name": "Bob", "department_id": 10, "salary": 4800},
    {"id": 3, "name": "Charlie", "department_id": 10, "salary": 5200},
    {"id": 4, "name": "David", "department_id": 20, "salary": 6000},
    {"id": 5, "name": "Eve", "department_id": 20, "salary": 5900}
]

The query will produce:

JSON
Copy
[
    {"department_id": 10, "salary": 4800},
    {"department_id": 20, "salary": 5900}
]
```

**Data:**

```json
{
    "employees": [
        {"id": 1, "name": "Alice", "department_id": 10, "salary": 5000},
        {"id": 2, "name": "Bob", "department_id": 10, "salary": 4800},
        {"id": 3, "name": "Charlie", "department_id": 10, "salary": 5200},
        {"id": 4, "name": "David", "department_id": 20, "salary": 6000},
        {"id": 5, "name": "Eve", "department_id": 20, "salary": 5900}
    ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is there at least onedepartment with more thanone unique salary?
5. Partition by department_id,rank salaries
6. Rank 2nd highest salary usingROW_NUMBER()
7. Select and return secondhighest salary per department
8. No second highest salary forthis department

