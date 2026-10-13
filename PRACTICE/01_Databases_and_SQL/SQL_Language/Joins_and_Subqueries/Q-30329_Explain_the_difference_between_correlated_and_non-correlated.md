---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between correlated and non-correlated placement of a subquery in HAVING and how it affects grouping cost.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Rewriting Subqueries as Joins](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/Rewriting_Subqueries_as_Joins.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, understanding the nuances between correlated and non-correlated subqueries within HAVING clauses is crucial for optimizing query performance. This answer will explain these concepts, their placement in HAVING, and how they affect grouping costs.

#### Triage
What the Question Is Asking

The question asks us to differentiate between using a correlated subquery vs. a non-correlated subquery within the HAVING clause of SQL queries. The focus is on understanding the implications for query performance, specifically in terms of grouping cost.

#### Assumptions
and Core Idea
Correlated subqueries depend on values from outer queries.
Non-correlated subqueries are evaluated separately and reused across all groups.
Both types can be placed within HAVING clauses to filter grouped results.

#### First Principles Solution
1. Correlated Subquery in HAVING

A correlated subquery in the HAVING clause is one that contains a reference to an aggregate function or column from the outer query's GROUP BY clause. The subquery is evaluated for each group individually, leading to higher grouping costs.

#### Example

SQL
SELECT department_id, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING (SELECT COUNT(*) FROM employees WHERE salary > 50000 AND department_id = e.department_id) > 1;
Here, the subquery (SELECT COUNT(*) FROM employees WHERE salary > 50000 AND department_id = e.department_id) is correlated because it references department_id.
This correlated subquery will be executed once for each group in the outer query.
2. Non-Correlated Subquery in HAVING

A non-correlated subquery does not reference any columns from the outer query and can be evaluated independently. The results are then used to filter groups after all groups have been aggregated.

#### Example

SQL
SELECT department_id, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;
In this case, the subquery (COUNT(*) > 10) is non-correlated and evaluated once for all groups.
Performance Impact
Correlated Subquery
Higher grouping cost due to per-group evaluation.
Can lead to slower query execution times, especially with large datasets or complex conditions in the subquery.
Non-Correlated Subquery
Lower grouping cost because it’s a single evaluation.
Generally faster and more efficient for filtering grouped results.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Using correlated subqueries within HAVING can significantly increase grouping costs, making queries slower. Conversely, non-correlated subqueries are generally more efficient but must be carefully designed to avoid complexity in the main query. Common mistakes include overusing correlated subqueries or neglecting their impact on performance.

#### Follow-up
questions might explore:

How can indexes improve the performance of both types of subqueries?
What alternative approaches exist for achieving similar filtering results without using subqueries?

**Comparison:**

| Type | Evaluation | Cost |
| --- | --- | --- |
| Correlated | Once per group | Higher |
| Non-Correlated | Once for all groups | Lower |

**Code:**

```sql
-- Example with correlated subquery in HAVING
SELECT department_id, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING (SELECT COUNT(*) FROM employees WHERE salary > 50000 AND department_id = e.department_id) > 1;

-- Example with non-correlated subquery in HAVING
SELECT department_id, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;
```

**Example input / output:**

```text
-- Input Query (Correlated Subquery in HAVING)
SELECT department_id, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING (SELECT COUNT(*) FROM employees WHERE salary > 50000 AND department_id = e.department_id) > 1;

-- Output for Input with Sample Data
+------------+--------------+
| department_id | avg_salary |
+------------+--------------+
| 1           | 52000.00    |
+------------+--------------+

-- Input Query (Non-Correlated Subquery in HAVING)
SELECT department_id, AVG(salary) as avg_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 10;

-- Output for Input with Sample Data
No rows returned because there are less than 10 employees in any department.
```

**Data:**

```json
{
  "employees": [
    {"employee_id": 1, "name": "John Doe", "salary": 52000, "department_id": 1},
    {"employee_id": 2, "name": "Jane Smith", "salary": 48000, "department_id": 1},
    {"employee_id": 3, "name": "Alice Johnson", "salary": 60000, "department_id": 2}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query Starts
4. Correlated Subquery EvaluatedPer Group
5. Is the subquery correlated?
6. Higher Grouping Cost, SlowerQuery
7. Lower Grouping Cost, FasterQuery

