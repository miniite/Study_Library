---
type: "practice"
difficulty: "Intermediate"
---

# How would you compute the difference between each employee's salary and the department average on the same row?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Top-N per Group](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to calculate the difference between an employee's salary and the average salary of their respective department. This can be achieved using SQL window functions. The key idea is that we can use a window function to compute the average salary within each department and then subtract it from the individual employee’s salary.

**Code:**

```sql
-- Sample SQL query using window functions
SELECT
    emp_id,
    emp_name,
    dept_id,
    salary,
    (salary - AVG(salary) OVER (PARTITION BY dept_id)) AS diff_from_dept_avg_salary
FROM
    employees;
```

**Example input / output:**

```text
-- Sample input query
SELECT
    emp_id,
    emp_name,
    dept_id,
    salary,
    (salary - AVG(salary) OVER (PARTITION BY dept_id)) AS diff_from_dept_avg_salary
FROM
    employees;

-- Expected output
emp_id | emp_name | dept_id | salary | diff_from_dept_avg_salary
-----------------------------------------
1      | Alice    | 10      | 7500   | -625.00                        (avg: 8375)
2      | Bob      | 10      | 8000   | 625.00                         (avg: 8375)
3      | Charlie  | 20      | 9000   | -250.00                        (avg: 9250)
4      | David    | 20      | 9500   | 250.00                         (avg: 9250)
```

**Data:**

```json
{
  "employees": [
    {"emp_id": 1, "emp_name": "Alice", "dept_id": 10, "salary": 7500},
    {"emp_id": 2, "emp_name": "Bob", "dept_id": 10, "salary": 8000},
    {"emp_id": 3, "emp_name": "Charlie", "dept_id": 20, "salary": 9000},
    {"emp_id": 4, "emp_name": "David", "dept_id": 20, "salary": 9500}
  ]
}
```

