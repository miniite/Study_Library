---
type: "practice"
difficulty: "Expert"
---

# Explain how recursive CTE cardinality estimates are usually wrong and how bad estimates cascade into poor join plans downstream.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Recursive CTEs and Materialization](<../../../../01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database setting, Common Table Expressions (CTEs) are powerful tools for breaking down complex queries into simpler, more manageable parts. Recursive CTEs specifically allow us to solve problems by repeatedly calling themselves until a base case is reached. However, cardinality estimates for recursive CTEs can often be incorrect, leading to suboptimal execution plans and performance issues.

To understand why these estimates are typically wrong and how they impact downstream operations, let's start from the basics of query optimization. The database optimizer uses statistics and heuristics to estimate the number of rows returned by each operation (cardinality) in a query plan. These estimates guide the choice of algorithms, such as nested loops or hash joins, which can significantly affect performance.

#### Triage
What the Question is Asking

The question delves into why cardinality estimates for recursive CTEs are frequently inaccurate and how these inaccuracies propagate through subsequent operations like joins. The interviewer is probing your understanding of:

Cardinality Estimation: How databases predict the number of rows returned by queries.
Recursive CTEs: How they work, especially in complex scenarios.
Query Optimization: The decision-making process behind choosing between different join strategies.

#### First-Principles Solution
Cardinality Estimates and Recursive CTEs

Cardinality estimates are often based on statistical sampling of the data. For non-recursive queries, these estimates can be reasonably accurate because they reflect the overall distribution of data. However, for recursive CTEs, the situation is more complex:

Initial Query Execution: The first iteration of a recursive CTE might return many rows, which could lead to an overestimate.
Subsequent Iterations: As the recursion progresses, the number of rows returned by each iteration can vary widely, making it difficult for the optimizer to predict accurately.

#### Example Scenario

#### Cons
ider a simple recursive CTE that counts all paths in a hierarchical structure (e.g., employee hierarchy):

SQL
WITH RECURSIVE EmployeePaths AS (
    SELECT emp_id, manager_id, 1 AS level FROM Employees WHERE manager_id IS NULL
    UNION ALL
    SELECT E.emp_id, E.manager_id, EP.level + 1 FROM Employees E JOIN EmployeePaths EP ON E.manager_id = EP.emp_id
)
SELECT * FROM EmployeePaths;

If the initial set of managers is large, the optimizer might overestimate the number of rows in each recursive step. This can lead to a less efficient join plan being chosen.

Impact on Join Plans

The cardinality estimates for recursive CTEs influence how joins are planned and executed. For example:

Nested Loop Joins: If the optimizer expects only a few rows from one side, it might choose nested loops instead of hash joins or merge joins.
Hash Joins and Merge Joins: These methods require more memory and disk space. Inaccurate estimates can lead to insufficient resources being allocated.

#### Closing Summary

Recursive CTEs are powerful but can introduce complexity in cardinality estimation. Inaccuracies in these estimates can cascade through the query execution, leading to poor join plans and overall suboptimal performance. Understanding this issue is crucial for database developers, especially when dealing with complex hierarchical data or recursive operations.

#### Common Mistakes/Follow-Ups
Over-reliance on Initial Estimates: Optimizers may not adapt well to changes in the number of rows returned by each iteration.
Optimistic Estimation: Expectations that recursive CTEs will always return small numbers, leading to suboptimal plans.
Lack of Detailed Analysis: Not thoroughly testing and validating cardinality estimates through actual execution or dynamic sampling.

Recursive CTEs require careful handling and validation to ensure accurate query performance.

**Comparison:**

| emp_id | manager_id | level |
|--------|-----------|-------|
| 1      | NULL      | 1     |
| 2      | 1         | 2     |
| 3      | 1         | 2     |
| 4      | 2         | 3     |
| 5      | 2         | 3     |
| 6      | 3         | 4     |

**Code:**

```sql
WITH RECURSIVE EmployeePaths AS (
    SELECT emp_id, manager_id, 1 AS level FROM Employees WHERE manager_id IS NULL
    UNION ALL
    SELECT E.emp_id, E.manager_id, EP.level + 1 FROM Employees E JOIN EmployeePaths EP ON E.manager_id = EP.emp_id
)
SELECT * FROM EmployeePaths;
```

**Example input / output:**

```

WITH RECURSIVE EmployeePaths AS ( SELECT emp_id, manager_id, 1 AS level FROM Employees WHERE manager_id IS NULL UNION ALL SELECT E.emp_id, E.manager_id, EP.level + 1 FROM Employees E JOIN EmployeePaths EP ON E.manager_id = EP.emp_id ) SELECT * FROM EmployeePaths;

CODE
Copy
Output:
```sql
| emp_id | manager_id | level |
|--------|-----------|-------|
|      1 |        NULL |     1 |
|      2 |          1 |     2 |
|      3 |          1 |     2 |
|      4 |          2 |     3 |
|      5 |          2 |     3 |
|      6 |          3 |     4 |
```

**Data:**

```sql
CREATE TABLE Employees (
    emp_id INT PRIMARY KEY,
    manager_id INT,
    name VARCHAR(50)
);

INSERT INTO Employees (emp_id, manager_id, name) VALUES
(1, NULL, 'CEO'),
(2, 1, 'CTO'),
(3, 1, 'CFO'),
(4, 2, 'Dev Manager'),
(5, 2, 'HR Manager'),
(6, 3, 'Finance Manager');
```

**Flow:**

1. Many Rows
2. Few Rows
3. Recursive CTE Initial Execution
4. Rows Returned?
5. Inaccurate Cardinality Estimate
6. Optimal Join Plan Chosen
7. Suboptimal Join Plan Chosen
8. Execution Performance Issues
9. Execution Efficiency

