---
type: "practice"
difficulty: "Basic"
---

# Explain scope and visibility of columns defined in one CTE when referenced by a later CTE in the same WITH clause.

**Difficulty:** Basic · **Levels:** SDE-1, SDE-1/2 · [↑ Recursive CTEs and Materialization](<../../../../01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In SQL, Common Table Expressions (CTEs) are a powerful feature that allow you to define temporary result sets and use them as tables within a single query. When using multiple CTEs in the same WITH clause, it's important to understand how columns defined in one CTE are visible or not in later CTEs. Essentially, each CTE is its own scope, but there are specific rules about column visibility.

#### Triaging the Question

The question is essentially asking you to explain the scoping and visibility of columns within a series of CTEs defined together. This involves understanding how SQL handles variable and column definitions across different parts of a query when using CTEs.

#### Assumptions
and Scope Rules
Assumption 1: You are working with standard SQL, as the behavior can vary slightly between different database systems.
Assumption 2: Each CTE in the WITH clause is executed independently but its results can be used by subsequent CTEs or the main query.

#### First Principles Solution

Defining a CTE:

A CTE is defined using the WITH keyword followed by a name and a SQL statement that returns a result set.

Visibility in Subsequent CTEs:

Columns defined in an earlier CTE are not automatically visible to later CTEs unless they are explicitly referenced or selected into a table.

#### Example Scenario
Let's consider the following query:

SQL
WITH cte1 AS (
   SELECT id, name FROM employees
),
cte2 AS (
   SELECT id, salary FROM cte1 WHERE id = 100
)
SELECT * FROM cte2;

Explanation:

In cte1, the columns id and name are defined.
In cte2, only id is selected, but not name.
The column salary in cte2 is derived from a WHERE clause that references id from cte1.

Column Visibility:

name from cte1 is not visible in cte2.
salary and id are defined in cte2 based on the results of cte1.

#### Closing Summary

In summary, columns defined in an earlier CTE within the same WITH clause are only directly visible if they are explicitly referenced or selected. Subsequent CTEs can derive new columns that are not necessarily visible from previous CTEs.

#### Common Mistakes
and Follow-Ups
Mistake 1: Assuming all columns from one CTE are automatically available in another.

#### Follow-Up
Question: How does this change if we use JOIN between CTEs?

**Code:**

```sql
WITH cte1 AS (
   SELECT id, name FROM employees
),
cte2 AS (
   SELECT id, salary FROM cte1 WHERE id = 100
)
SELECT * FROM cte2;
```

**Example input / output:**

```text
WITH cte1 AS (
   SELECT id, name FROM employees
),
cte2 AS (
   SELECT id, salary FROM cte1 WHERE id = 3
)
SELECT * FROM cte2;

-- Output:
+----+--------+
| id | salary |
+----+--------+
|  3 | 7000   |
+----+--------+
```

**Data:**

```json
{
  "employees": [
    { "id": 1, "name": "Alice", "salary": 5000 },
    { "id": 2, "name": "Bob", "salary": 6000 },
    { "id": 3, "name": "Charlie", "salary": 7000 }
  ]
}
```

