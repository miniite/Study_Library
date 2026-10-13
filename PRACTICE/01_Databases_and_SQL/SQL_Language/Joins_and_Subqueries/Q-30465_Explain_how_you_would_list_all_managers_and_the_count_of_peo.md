---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would list all managers and the count of people reporting directly to them.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Subquery Practice Problems](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Subquery_Practice_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the task of listing all managers along with the count of people reporting directly to each, we need to leverage SQL joins and subqueries effectively. The core idea is to use a self-join on the employee table to find direct reports and then aggregate the results.

#### Triaging the Question

The question asks us to list managers alongside the number of their direct reports. This involves:

Identifying Managers: Typically, a manager will have employees directly reporting to them.
Counting Direct Reports: We need to count how many such employees exist for each manager.

#### Assumptions
and First Principles
Assume we have an employees table with columns: emp_id, name, and manager_id.
The manager_id is the emp_id of the employee who reports directly to this employee.
We want a list where each row contains the manager's name and the count of their direct reports.
Step-by-Step Solution
Identify Managers: A manager will have at least one entry in the table with another employee as their manager_id.
Count Direct Reports: For each manager, count how many employees directly report to them by checking where their emp_id is the manager_id of someone else.
SQL Solution
SQL
SELECT
    e1.name AS manager_name,
    COUNT(e2.emp_id) AS direct_report_count
FROM
    employees e1
JOIN
    employees e2 ON e1.emp_id = e2.manager_id
GROUP BY
    e1.emp_id, e1.name;
Explanation
Self-Join: We join the employees table with itself on the condition that the manager's ID (e1.emp_id) matches the employee's ID where they are a direct report (e2.manager_id = e1.emp_id).
Group By: We group by emp_id and name to ensure we get one row per manager.
Count: The COUNT(e2.emp_id) gives us the number of direct reports for each manager.

#### Closing Summary

This solution effectively uses a self-join with subquery logic to find managers and count their direct reports. It's essential to consider edge cases like employees who have no managers or situations where there might be cycles in the hierarchy, though they are not common in typical organizational structures.

#### Common Mistakes/Follow-Ups
Incorrect Join Condition: Ensure you use e2.manager_id = e1.emp_id to correctly identify direct reports.
Handling No Managers: If an employee has no manager (e.g., the CEO), they won't be included in the self-join and thus might need a separate query or logic to include them if necessary.
Hierarchical Cycles: In rare cases, ensure there are no hierarchical cycles that could affect the join.

**Comparison:**

```
emp_id	name	manager_id
1	Alice	NULL
2	Bob	1
3	Charlie	1
4	David	2
5	Eve	3
```

**Code:**

```sql
SELECT
    e1.name AS manager_name,
    COUNT(e2.emp_id) AS direct_report_count
FROM
    employees e1
JOIN
    employees e2 ON e1.emp_id = e2.manager_id
GROUP BY
    e1.emp_id, e1.name;
```

**Example input / output:**

```text
Input:
No specific input needed as the SQL statement can be executed directly.

Output:
+------------+----------------+
| manager_name | direct_report_count |
+------------+----------------+
| Alice       | 2                 |
+------------+----------------+
```

**Data:**

```sql
CREATE TABLE employees (
    emp_id INT,
    name VARCHAR(255),
    manager_id INT
);
INSERT INTO employees (emp_id, name, manager_id) VALUES
(1, 'Alice', NULL),  -- CEO
(2, 'Bob', 1),       -- Reports to Alice
(3, 'Charlie', 1),   -- Reports to Alice
(4, 'David', 2),     -- Reports to Bob
(5, 'Eve', 3);       -- Reports to Charlie
```

**Flow:**

1. Employees Table
2. Join Condition
3. Self-Join on manager_id
4. GROUP BY emp_id, name
5. Count Direct Reports

