---
type: "practice"
difficulty: "Intermediate"
---

# Write a query to compute the second-highest salary per department (not overall).

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Top-N per Group](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write an SQL query that computes the second-highest salary for each department. The key challenge here is to handle potential ties in salaries within a department and ensure we correctly identify the second-highest distinct value.

#### Triage

The question is asking us to find the second-highest salary per department, which means:

We need to group by departments.
For each department, we need to order the salaries and determine the second-highest unique value.

#### First-Principles Solution

To achieve this, we can use window functions in SQL. Specifically, we'll leverage the DENSE_RANK() function to assign a rank to each salary within its department, then filter out the top two ranks to identify the second-highest salary.

Here’s the step-by-step breakdown:

Rank Salaries: Use DENSE_RANK() to order salaries within each department and assign a rank.
Filter Ranks: Select rows where the rank is 2 (indicating the second-highest salary).
SQL Query
SQL
WITH RankedSalaries AS (
    SELECT
        department_id,
        salary,
        DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as salary_rank
    FROM
        employees
)
SELECT
    department_id,
    MAX(CASE WHEN salary_rank = 2 THEN salary END) as second_highest_salary
FROM
    RankedSalaries
GROUP BY
    department_id;

#### Closing Summary
and Common Mistakes

This solution uses window functions to rank salaries within each department. By filtering the ranks, we can identify the second-highest unique salary for each group.

#### Common Mistakes

Misuse of RANK() vs DENSE_RANK(): Using RANK() instead of DENSE_RANK() could skip over tied values, leading to an incorrect result.
Incorrect Grouping and Ranking: Not properly partitioning the data by department can lead to incorrect rank assignments.
Lack of MAX Filter: Forgetting to use a filter like MAX(CASE...) ensures that only one value is returned per department.

#### Example
Data
JSON
{
    "employees": [
        {"id": 1, "name": "Alice", "salary": 5000, "department_id": 1},
        {"id": 2, "name": "Bob", "salary": 6000, "department_id": 1},
        {"id": 3, "name": "Charlie", "salary": 7000, "department_id": 1},
        {"id": 4, "name": "David", "salary": 5000, "department_id": 2},
        {"id": 5, "name": "Eve", "salary": 6000, "department_id": 2}
    ]
}
Worked Input->Output

For the sample data:

SQL
WITH RankedSalaries AS (
    SELECT
        department_id,
        salary,
        DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as salary_rank
    FROM
        employees
)
SELECT
    department_id,
    MAX(CASE WHEN salary_rank = 2 THEN salary END) as second_highest_salary
FROM
    RankedSalaries
GROUP BY
    department_id;

Output:

department_id	second_highest_salary
1	6000
2	6000

**Code:**

```sql
WITH RankedSalaries AS (
    SELECT
        department_id,
        salary,
        DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as salary_rank
    FROM
        employees
)
SELECT
    department_id,
    MAX(CASE WHEN salary_rank = 2 THEN salary END) as second_highest_salary
FROM
    RankedSalaries
GROUP BY
    department_id;
```

**Example input / output:**

```sql
WITH RankedSalaries AS (
    SELECT
        department_id,
        salary,
        DENSE_RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) as salary_rank
    FROM
        employees
)
SELECT
    department_id,
    MAX(CASE WHEN salary_rank = 2 THEN salary END) as second_highest_salary
FROM
    RankedSalaries
GROUP BY
    department_id;
department_id	second_highest_salary
1	6000
2	6000
```

**Data:**

```json
{
    "employees": [
        {"id": 1, "name": "Alice", "salary": 5000, "department_id": 1},
        {"id": 2, "name": "Bob", "salary": 6000, "department_id": 1},
        {"id": 3, "name": "Charlie", "salary": 7000, "department_id": 1},
        {"id": 4, "name": "David", "salary": 5000, "department_id": 2},
        {"id": 5, "name": "Eve", "salary": 6000, "department_id": 2}
    ]
}
```

