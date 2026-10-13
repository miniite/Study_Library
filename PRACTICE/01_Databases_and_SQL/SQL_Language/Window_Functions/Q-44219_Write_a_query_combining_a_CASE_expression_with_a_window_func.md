---
type: "practice"
difficulty: "Intermediate"
---

# Write a query combining a CASE expression with a window function to label the top earner in each department.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Top-N per Group](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to combine a CASE statement with a window function to identify the top earner in each department. The query will involve ranking employees based on their salary within each department and then using the CASE expression to label the top earner.

#### Triage

The question is asking us to write an SQL query that uses both a CASE expression and a window function to determine which employee in each department has the highest salary. The goal here is not just to identify the maximum salary but also to add a column indicating who holds this position within their respective department.

#### First-Principles Solution
Ranking Employees by Salary: We will use the RANK() window function to rank employees in each department based on their salaries.
Labeling Top Earners: Using a CASE expression, we will label the employee with the highest salary as "Top Earner".
SQL Query
SQL
SELECT
    id,
    name,
    department_id,
    salary,
    CASE
        WHEN RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) = 1 THEN 'Top Earner'
        ELSE NULL
    END AS top_earner_label
FROM
    employees;
Explanation
Window Function (RANK()): We partition the data by department_id and order it by salary in descending order. The RANK() function assigns a rank to each row, starting with 1 for the highest salary within each department.
CASE Expression: We check if the current employee's rank is 1 (i.e., they have the highest salary). If so, we label them as "Top Earner"; otherwise, we return NULL.

#### Closing Summary

This query effectively combines a window function with a CASE expression to identify and label the top earner in each department. The RANK() function ensures that we correctly rank employees based on their salaries within their departments, while the CASE expression allows us to assign labels accordingly.

#### Common Mistakes/Follow-Ups
Mistake: Using ROW_NUMBER() instead of RANK(). While both can be used, ROW_NUMBER() would give a unique rank for each row regardless of ties, which might not be what we want if there are multiple employees with the same highest salary.

#### Follow-Up
How to handle departments with no employees or employees having the same salary? In such cases, you may need to decide on how to label or handle these scenarios.

**Comparison:**

| id | name   | department_id | salary | top_earner_label |
|----|--------|--------------|--------|-----------------|
| 2  | Bob    | 101          | 6000   | Top Earner      |
| 3  | Charlie| 101          | 6000   | NULL            |
| 4  | David  | 102          | 7000   | Top Earner      |

**Code:**

```sql
SELECT
    id,
    name,
    department_id,
    salary,
    CASE
        WHEN RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) = 1 THEN 'Top Earner'
        ELSE NULL
    END AS top_earner_label
FROM
    employees;
```

**Example input / output:**

```text
-- Sample Input (assuming the above JSON data)
SELECT
    id,
    name,
    department_id,
    salary,
    CASE
        WHEN RANK() OVER (PARTITION BY department_id ORDER BY salary DESC) = 1 THEN 'Top Earner'
        ELSE NULL
    END AS top_earner_label
FROM
    employees;

-- Expected Output
+----+--------+-----------+---------+----------------+
| id | name   | department_id | salary | top_earner_label |
+----+--------+-----------+---------+----------------+
| 2  | Bob    | 101        | 6000   | Top Earner      |
| 3  | Charlie| 101        | 6000   | NULL            |
| 4  | David  | 102        | 7000   | Top Earner      |
+----+--------+-----------+---------+----------------+
```

**Data:**

```json
{
  "employees": [
    { "id": 1, "name": "Alice", "department_id": 101, "salary": 5000 },
    { "id": 2, "name": "Bob", "department_id": 101, "salary": 6000 },
    { "id": 3, "name": "Charlie", "department_id": 101, "salary": 6000 },
    { "id": 4, "name": "David", "department_id": 102, "salary": 7000 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start Query
4. RANK() over (PARTITION BYdepartment_id ORDER BYsalary DESC)
5. Check if RANK = 1
6. Label as 'Top Earner'
7. NULL
8. Output row with label

