---
type: "practice"
difficulty: "Intermediate"
---

# How would you find, for each employee, the salary of the highest-paid person in their department using MAX() OVER?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Top-N per Group](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

SQL
-- Input Schema (Example Data)
CREATE TABLE employees (
    id INT,
    name VARCHAR(100),
    department VARCHAR(50),
    salary DECIMAL
);

INSERT INTO employees (id, name, department, salary) VALUES
(1, 'Alice', 'Sales', 60000),
(2, 'Bob', 'Marketing', 75000),
(3, 'Charlie', 'HR', 80000),
(4, 'Diana', 'Sales', 90000),
(5, 'Eve', 'Engineering', 120000);
SQL
-- Output Query Result
SELECT id, name, department, salary,
       MAX(salary) OVER (PARTITION BY department) AS highest_salary_in_dept
FROM employees;

Output:

id	name	department	salary	highest_salary_in_dept
1	Alice	Sales	60000	90000
2	Bob	Marketing	75000	75000
3	Charlie	HR	80000	80000
4	Diana	Sales	90000	90000
5	Eve	Engineering	120000	120000

