---
type: "practice"
difficulty: "Basic"
---

# How would you show, for each employee, how their salary compares to the maximum salary in their team?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Top-N per Group](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Top-N_per_Group.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer this question, we need to calculate the highest salary within each employee's team and compare it with each individual's salary. This involves using window functions in SQL, which allow us to perform calculations across a set of table rows related to a single row.

Firstly, let’s assume our data is stored in an employees table that contains fields such as employee_id, team_id, salary. Our goal is to generate a result showing each employee's salary and the maximum salary within their respective team.

Here’s how we can break down the solution step by step:

Identify the relevant columns: We need to focus on employee_id, team_id, and salary.
Window function setup: Use the MAX() window function to find the highest salary in each team.
Calculate the comparison: Subtract or compare the employee's salary with the maximum salary within their team.

Let’s write out the SQL query step by step:

SQL
SELECT
    e.employee_id,
    e.team_id,
    e.salary,
    MAX(e2.salary) OVER (PARTITION BY e.team_id) AS max_salary_in_team
FROM
    employees e;

This query uses a window function MAX() to find the maximum salary (max_salary_in_team) within each team defined by team_id. The result set will include the employee’s ID, their salary, and the highest salary in their respective team.

**Comparison:**

| employee_id | team_id | salary |
|-------------|---------|--------|
| 1           | 101     | 6000   |
| 2           | 101     | 7000   |
| 3           | 102     | 5500   |
| 4           | 102     | 6500   |
| 5           | 103     | 8000   |
| employee_id | team_id | salary | max_salary_in_team |
|-------------|---------|--------|--------------------|
| 1           | 101     | 6000   | 7000               |
| 2           | 101     | 7000   | 7000               |
| 3           | 102     | 5500   | 6500               |
| 4           | 102     | 6500   | 6500               |
| 5           | 103     | 8000   | NULL               |

**Code:**

```sql
SELECT
    e.employee_id,
    e.team_id,
    e.salary,
    MAX(e2.salary) OVER (PARTITION BY e.team_id) AS max_salary_in_team
FROM
    employees e;
```

**Example input / output:**

| employee_id | team_id | salary | max_salary_in_team |
|-------------|---------|--------|--------------------|
| 1           | 101     | 6000   | 7000               |
| 2           | 101     | 7000   | 7000               |
| 3           | 102     | 5500   | 6500               |
| 4           | 102     | 6500   | 6500               |
| 5           | 103     | 8000   | NULL               |

**Data:**

```json
{
  "employees": [
    {"employee_id": 1, "team_id": 101, "salary": 6000},
    {"employee_id": 2, "team_id": 101, "salary": 7000},
    {"employee_id": 3, "team_id": 102, "salary": 5500},
    {"employee_id": 4, "team_id": 102, "salary": 6500},
    {"employee_id": 5, "team_id": 103, "salary": 8000}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is the employee in a team?
5. Get the maximum salary of theemployee's team
6. Compare the employee'ssalary with themax_salary_in_team
7. Output the result(employee_id, salary,max_salary_in_team)
8. Handle employees not in ateam (optional)

