---
type: "practice"
difficulty: "Deep"
---

# What does the ANY (or SOME) operator do with a subquery? Give an example.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Joins vs Subqueries and Join Types](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Joins_vs_Subqueries_and_Join_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The ANY or SOME operator in SQL is used to compare a value from a column of a subquery against values returned by that subquery. Essentially, it checks if any of the elements in the subquery match the condition specified with the main query. This operator can be very useful for filtering records based on conditions derived from another table or dataset.

To understand this better, let's break down how ANY works and then provide an example to illustrate its usage.

**Comparison:**

| employee_id | name   | salary  | department_id |
|-------------|--------|---------|---------------|
| 1           | Alice  | 5000    | 1             |
| 2           | Bob    | 6000    | 1             |
| 3           | Charlie| 7000    | 2             |
| 4           | Dave   | 8000    | 2             |
| 5           | Eve    | 9000    | 3             |
| department_id | average_salary |
|---------------|----------------|
| 1             | 5500           |
| 2             | 7500           |
| 3             | 9000           |

**Code:**

```sql
-- Example Scenario:
-- We have a database of Employees with Departments. Each Employee is assigned to one Department.
-- We want to find employees who earn more than the average salary in their department.

SELECT employee_id, name, salary
FROM employees e1
WHERE salary > ANY (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.department_id = e1.department_id
);
```

**Example input / output:**

```text
Input:
- The main query is selecting employees who earn more than the average salary in their department.
- The subquery calculates the average salary for each department.

Output:
| employee_id | name   | salary  |
|-------------|--------|---------|
| 3           | Charlie| 7000    |
| 4           | Dave   | 8000    |
| 5           | Eve    | 9000    |
```

**Data:**

```plaintext
Table: employees
| employee_id | name   | salary  | department_id |
|-------------|--------|---------|---------------|
| 1           | Alice  | 5000    | 1             |
| 2           | Bob    | 6000    | 1             |
| 3           | Charlie| 7000    | 2             |
| 4           | Dave   | 8000    | 2             |
| 5           | Eve    | 9000    | 3             |
```

**Flow:**

1. Employees Table
2. Salary > ANY (SELECTAVG(salary) FROM EmployeesWHERE department_id =E.department_id)
3. Charlie, Dave, Eve

