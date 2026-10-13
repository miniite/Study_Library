---
type: "practice"
difficulty: "Deep"
---

# Write a query to find departments where the average salary exceeds the company-wide average salary.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Aggregate Precision and Types](<../../../../01_Databases_and_SQL/SQL_Language/Aggregation/Aggregate_Precision_and_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To solve this problem, we need to write a SQL query that calculates the average salary per department and compares it with the overall company-wide average salary. The key idea is to use aggregate functions like AVG() to compute these averages. Here’s a step-by-step breakdown of how to approach this:

Calculate the Company-Wide Average Salary: First, we need to determine what the average salary for all employees in the company is.
Compute Departmental Average Salaries: Next, we calculate the average salary for each department.
Compare and Filter Departments: Finally, we filter out departments where the departmental average salary exceeds the company-wide average.

**Comparison:**

| department_id | department_name | dept_avg_salary | avg_salary |
|---------------|-----------------|-----------------|------------|
| 10            | Marketing       | 5500.0          | 5833.33    |
| 20            | Sales           | 6500.0          | 6200.0     |
| 30            | Finance         | 6500.0          | 6200.0     |

**Code:**

```sql
-- Step 1: Calculate the company-wide average salary
WITH overall_avg_salary AS (
    SELECT AVG(salary) AS avg_salary
    FROM employees
)

-- Step 2 and 3: Compute departmental average salaries and compare with overall average
SELECT
    d.department_id,
    d.department_name,
    AVG(e.salary) OVER (PARTITION BY e.department_id) AS dept_avg_salary,
    o.avg_salary
FROM
    departments d
JOIN
    employees e ON d.department_id = e.department_id
CROSS JOIN
    overall_avg_salary o
WHERE
    AVG(e.salary) > (SELECT avg_salary FROM overall_avg_salary)
GROUP BY
    d.department_id, d.department_name, o.avg_salary;
```

**Example input / output:**

| department_id | department_name | dept_avg_salary | avg_salary |
|---------------|-----------------|-----------------|------------|
| 20            | Sales           | 6500.0          | 6200.0     |
| 30            | Finance         | 6500.0          | 6200.0     |

**Data:**

```json
{
  "departments": [
    {"department_id": 10, "department_name": "Marketing"},
    {"department_id": 20, "department_name": "Sales"},
    {"department_id": 30, "department_name": "Finance"}
  ],
  "employees": [
    {"employee_id": 101, "name": "Alice", "salary": 5000, "department_id": 10},
    {"employee_id": 102, "name": "Bob", "salary": 6000, "department_id": 10},
    {"employee_id": 103, "name": "Charlie", "salary": 5500, "department_id": 20},
    {"employee_id": 104, "name": "Dave", "salary": 7000, "department_id": 20},
    {"employee_id": 105, "name": "Eve", "salary": 6500, "department_id": 30}
  ]
}
```

**Flow:**

1. FilteringDepartmentsWithHigherSalary
2. DepartmentalAverageCalculation
3. OverallAverageCalculation
4. Start
5. Calculate Overall AverageSalary
6. Compute DepartmentalAverage Salaries
7. Filter Departments with HigherAvg Salary
8. End
9. Calculate SUM of All Salaries
10. Divide by Total Number ofEmployees
11. Gather Employee Details byDepartment
12. Average Salary perDepartment
13. Compare Departmental Avgwith Overall Average
14. Select Departments whereDeptAvg > OverallAvg

