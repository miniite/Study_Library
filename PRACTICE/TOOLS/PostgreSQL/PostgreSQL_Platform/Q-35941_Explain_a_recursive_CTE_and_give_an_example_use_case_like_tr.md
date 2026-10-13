---
type: "practice"
difficulty: "Deep"
---

# Explain a recursive CTE and give an example use case like traversing an org hierarchy.

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized CTEs](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Recursive Common Table Expressions (CTEs) are a powerful feature in PostgreSQL that allow you to perform operations on sets of rows using self-referencing queries. Essentially, they enable you to write SQL code that recursively processes data until a certain condition is met or the recursion stops. In this answer, we'll dive into what recursive CTEs are, how they work, and provide an example use case involving traversing an organization hierarchy.

#### Triaging the Question

The question asks for an explanation of recursive CTEs in PostgreSQL and provides an example use case to traverse an organizational hierarchy. The core idea here is understanding how recursive queries can be used to process hierarchical data efficiently. Assumptions include familiarity with basic SQL concepts such as joins, subqueries, and aggregate functions.

#### First-Principles Solution

A recursive CTE consists of two parts: the common table expression itself and its recursive part. The common part defines an initial set of rows from which the recursion starts. The recursive part then extends this set by querying for additional rows based on some condition. PostgreSQL supports both "WITH RECURSIVE" and "WITH" clauses, but we'll focus on "WITH RECURSIVE" as it's more commonly used in practice.

Let’s take an example where you have a table employees with columns id, name, manager_id, and department. The goal is to list all employees along with their managers up to the top manager (e.g., CEO).

#### Example
Use Case: Traversing Organization Hierarchy
SQL
WITH RECURSIVE employee_hierarchy AS (
    -- Initial query: find all employees and their immediate manager
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    -- Recursive part: get the managers of the current set of employees
    SELECT e.id, e.name, e.manager_id, eh.level + 1 AS level
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy;

In this example:

The employee_hierarchy CTE is defined as a recursive part.
Initially, it selects all top-level managers (those with no manager).
Then, for each employee in the hierarchy, it joins with itself to find their direct and indirect managers.
The level column tracks how many levels deep we are.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Recursive CTEs are a versatile tool for handling hierarchical data in PostgreSQL. They allow you to define complex relationships between rows in an elegant way, making the code more readable and maintainable. However, it's crucial to ensure that your recursion terminates properly to avoid infinite loops.

Common Pitfalls:

Forgetting to include a termination condition.
Not setting appropriate level or other tracking columns, leading to uncontrolled recursion.
Failing to test with large datasets to understand performance implications.
Depth + Language Match the Level

Given that this is an advanced question requiring deep understanding, we've explored both theoretical and practical aspects. We've discussed how recursive CTEs work in PostgreSQL, provided a detailed example, and highlighted common pitfalls. This aligns well with the level of a Fullstack SDE-2/3 or Backend SDE-2/3 who would be expected to handle such complex queries.

Final Thoughts

Recursive CTEs can significantly simplify hierarchical data processing tasks. However, they should be used judiciously, considering performance and potential infinite loops.

**Comparison:**

```
CEO
Manager A
Employee X
Employee Y
```

**Code:**

```sql
WITH RECURSIVE employee_hierarchy AS (
    SELECT id, name, manager_id, 1 as level
    FROM employees
    WHERE manager_id IS NULL

    UNION ALL

    SELECT e.id, e.name, e.manager_id, eh.level + 1 AS level
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.id
)
SELECT * FROM employee_hierarchy;
```

**Example input / output:**

```text
+----+---------+------------+-------+
| id | name    | manager_id | level |
+----+---------+------------+-------+
| 1  | CEO     | NULL       | 1     |
| 2  | Manager A | 1         | 2     |
| 3  | Employee X | 2        | 3     |
| 4  | Employee Y | 3        | 4     |
+----+---------+------------+-------+
```

**Data:**

```json
{
  "employees": [
    {"id": 1, "name": "CEO", "manager_id": null},
    {"id": 2, "name": "Manager A", "manager_id": 1},
    {"id": 3, "name": "Employee X", "manager_id": 2},
    {"id": 4, "name": "Employee Y", "manager_id": 3}
  ]
}
```

**Flow:**

1. +---------------------+
2. | Employees Table     |
3. +---------------------+
4. |
5. v
6. +---------------------+       +---------------------+
7. | CEO                 | ---->  | Manager A           |
8. +---------------------+       +---------------------+
9. |                       |
10. v                       v
11. +---------------------+       +---------------------+
12. | Employee X          | ---->  | Employee Y          |
13. +---------------------+       +---------------------+

