---
type: "practice"
difficulty: "Deep"
---

# How would a recursive CTE walk an employee-manager hierarchy and compute each employee's depth level?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Recursive CTEs](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Recursive_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To walk an employee-manager hierarchy using a recursive Common Table Expression (CTE) in SQL and calculate each employee’s depth level, we'll need to understand how CTEs work and some common pitfalls. Let's break down the problem step by step.

First, let's triage what the question is really asking:

What the Question is Asking: The interviewer wants you to demonstrate your understanding of recursive SQL queries using CTEs.

#### Assumptions
We will assume a table structure with employees and their managers, typically represented by an employees table containing columns like employee_id, manager_id, and possibly a depth_level.
Objective: Compute the depth level for each employee in the hierarchy.

Now, let's reason through this problem from first principles:

Table Structure:

We assume we have an employees table with the following structure:
SQL
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    manager_id INT,
    depth_level INT DEFAULT 0 -- Initialize to 0 for root nodes
);

Recursive CTE Logic:

The CTE will start from a base case (root node) and iteratively build the hierarchy by adding subordinates.
For each employee, we'll calculate their depth level as one plus the maximum depth level of all their direct reports.

SQL Implementation:

SQL
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: find root nodes (employees with no manager)
    SELECT
        employee_id,
        0 AS depth_level,
        NULL AS parent_id
    FROM
        employees
    WHERE
        manager_id IS NULL

    UNION ALL

    -- Recursive step: add direct reports to the hierarchy and increment their depth levels
    SELECT
        e.employee_id,
        eh.depth_level + 1 AS depth_level,
        e.manager_id AS parent_id
    FROM
        employees e
    JOIN
        employee_hierarchy eh ON e.manager_id = eh.employee_id
)
-- Final output: list all employees with their respective depth levels
SELECT * FROM employee_hierarchy;

Explanation of the CTE:

Base Case: We start by selecting all root nodes (employees who have no manager).
Recursive Step: For each employee in the hierarchy, we find their direct reports and increment their depth_level by 1.
The recursion stops naturally when there are no more employees to add.

#### Common Mistakes

Failing to handle circular references (where an employee is their own manager) or infinite loops.
Not initializing the depth level for root nodes correctly, leading to incorrect results.
Using non-recursive methods without understanding how CTEs work can result in inefficient solutions.

**Comparison:**

| employee_id | depth_level | parent_id |
|-------------|-------------|-----------|
| 1           | 0           | NULL      |
| 2           | 1           | 1         |
| 3           | 1           | 1         |
| 4           | 2           | 2         |
| 5           | 2           | 2         |
| 6           | 3           | 3         |

**Code:**

```sql
WITH RECURSIVE employee_hierarchy AS (
    SELECT
        employee_id,
        0 AS depth_level,
        NULL AS parent_id
    FROM
        employees
    WHERE
        manager_id IS NULL

    UNION ALL

    SELECT
        e.employee_id,
        eh.depth_level + 1 AS depth_level,
        e.manager_id AS parent_id
    FROM
        employees e
    JOIN
        employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy;
```

**Example input / output:**

```

WITH RECURSIVE employee_hierarchy AS ( SELECT employee_id, 0 AS depth_level, NULL AS parent_id FROM employees WHERE manager_id IS NULL

CODE
Copy
UNION ALL

SELECT
    e.employee_id,
    eh.depth_level + 1 AS depth_level,
    e.manager_id AS parent_id
FROM
    employees e
JOIN
    employee_hierarchy eh ON e.manager_id = eh.employee_id

) SELECT * FROM employee_hierarchy;

CODE
Copy
Output:
```sql
employee_id | depth_level | parent_id
-----------|-------------|-----------
1          | 0           | NULL
2          | 1           | 1
3          | 1           | 1
4          | 2           | 2
5          | 2           | 2
6          | 3           | 3
```

**Data:**

```sql
-- Example data
INSERT INTO employees (employee_id, manager_id) VALUES
(1, NULL), -- Root node
(2, 1),
(3, 1),
(4, 2),
(5, 2),
(6, 3);
```

**Flow:**

1. Yes
2. No
3. Root Nodes
4. Are there more subordinates?
5. Add Subordinates andIncrement Depth
6. Determine Parent-ChildRelationship
7. End Recursion
8. Final Hierarchy

