---
type: "practice"
difficulty: "Deep"
---

# A join to a view is slow because the view has DISTINCT — how does that block predicate pushdown and how do you restructure it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ EXISTS Semi-joins and Predicate Placement](<../../../../01_Databases_and_SQL/SQL_Language/Joins_and_Subqueries/EXISTS_Semi-joins_and_Predicate_Placement.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand why DISTINCT in a view can cause join performance problems and how to mitigate them by restructuring the query or view definition.

#### Triaging the Question

The question revolves around optimizing a slow join operation involving a view that includes a DISTINCT clause. The core of the problem is that predicate pushdown, which allows conditions from the outer query to be pushed into subqueries (like views), is being blocked by the presence of DISTINCT. We need to explore how this affects performance and identify ways to restructure the query or view definition.

Core Concept

Predicate Pushdown: This technique involves pushing down predicates (conditions) from an outer query into a subquery. When predicate pushdown works, it can significantly improve performance by reducing the amount of data processed in the inner queries.

Distinct Clause: A DISTINCT clause ensures that each row returned is unique, which can be useful for removing duplicates but also introduces overhead due to sorting and additional memory usage.

#### First-Principles Solution

Understanding the Blockage:

When a view contains a DISTINCT clause, it may force the database engine to materialize the result set before applying outer query filters (predicates).
This can prevent predicate pushdown because the distinct operation is applied after the join, rather than being pushed down.

Alternative Approaches:

Remove DISTINCT in View: If possible, remove the DISTINCT clause from the view and incorporate it into the main query.
Use a Common Table Expression (CTE): Introduce a CTE to handle the distinct operation separately, allowing predicate pushdown on the join.

#### Example
Solution:

SQL
-- Original Slow Query with View and DISTINCT
SELECT *
FROM orders o
JOIN (
    SELECT DISTINCT customer_id, name
    FROM customers c
) c
ON o.customer_id = c.customer_id;

-- Optimized Query Using CTE
WITH distinct_customers AS (
    SELECT customer_id, name
    FROM customers
    WHERE name LIKE '%Smith%'  -- Example predicate pushed down
)
SELECT *
FROM orders o
JOIN distinct_customers dc
ON o.customer_id = dc.customer_id;

#### Closing Summary
and Common Mistakes

By restructuring the query to use a CTE, we can push the WHERE predicate from the main query into the CTE, which is more efficient. However, be cautious about the impact on query complexity and maintainability.

#### Common Mistakes

Overlooking the benefits of predicate pushdown.
Failing to recognize that DISTINCT clauses can block useful optimizations.
Not considering alternative query structures like CTEs or derived tables.
Final Thoughts

Always consider the trade-offs between simplicity and performance. In some cases, a more complex but optimized query might be necessary for better performance, especially in large-scale systems where every millisecond matters.

**Comparison:**

| Column | Type |
|--------|------|
| order_id | INT |
| customer_id | INT |
| Column | Type |
|--------|------|
| customer_id | INT |
| name | VARCHAR(255) |

**Code:**

```sql
-- Original Slow Query with View and DISTINCT
SELECT *
FROM orders o
JOIN (
    SELECT DISTINCT customer_id, name
    FROM customers c
) c
ON o.customer_id = c.customer_id;

-- Optimized Query Using CTE
WITH distinct_customers AS (
    SELECT customer_id, name
    FROM customers
    WHERE name LIKE '%Smith%'
)
SELECT *
FROM orders o
JOIN distinct_customers dc
ON o.customer_id = dc.customer_id;
```

**Example input / output:**

```text
Input:
  Orders Table: SELECT * FROM orders o JOIN (SELECT DISTINCT customer_id, name FROM customers c) c ON o.customer_id = c.customer_id;

Output:
  Optimized Query using CTE:

WITH distinct_customers AS (
    SELECT customer_id, name
    FROM customers
    WHERE name LIKE '%Smith%'
)
SELECT *
FROM orders o
JOIN distinct_customers dc
ON o.customer_id = dc.customer_id;
```

**Data:**

```sql
-- Sample Data Schemas
CREATE TABLE orders (order_id INT, customer_id INT);
CREATE TABLE customers (customer_id INT, name VARCHAR(255));
```

**Flow:**

1. No
2. Yes
3. Original Query
4. Predicate Pushdown?
5. Materialize View
6. Slow Performance
7. Push Predicate Down

