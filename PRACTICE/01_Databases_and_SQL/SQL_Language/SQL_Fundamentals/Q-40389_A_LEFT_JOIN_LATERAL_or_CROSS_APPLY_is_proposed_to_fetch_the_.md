---
type: "practice"
difficulty: "Expert"
---

# A LEFT JOIN LATERAL (or CROSS APPLY) is proposed to fetch the top-3 recent orders per customer in one query. When does a lateral join beat a window-function approach, and what are its indexing requirements?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Attribution with Window Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Attribution_with_Window_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the question of when a LEFT JOIN LATERAL (or CROSS APPLY) beats a window function approach for fetching top-3 recent orders per customer, we need to consider several factors including query performance, indexing requirements, and data distribution. The core idea is that lateral joins can provide more flexibility in generating correlated subqueries or derived tables on-the-fly, which might not be as straightforward with window functions.

#### Triaging the Question
Core of the Question: When would using a LEFT JOIN LATERAL (or CROSS APPLY) outperform a window function approach?

#### Assumptions
We are dealing with a relational database that supports these types of joins.
The goal is to fetch the top-3 recent orders for each customer.
Factors to Consider: Query performance, indexing strategy, and data distribution.

#### First Principles Solution
When Lateral Joins Outperform Window Functions

Flexibility in Joining Derived Tables:

Lateral joins allow you to generate a derived table (subquery) for each row of the outer query result set.
This flexibility can be advantageous when the subquery logic is complex or cannot easily fit into a window function.

Complex Query Logic:

If the top-3 recent orders calculation involves additional correlated logic, such as filtering based on order details (e.g., status, payment method), then a lateral join might be more natural.
For example, you could write a complex subquery that determines which rows to include in the top-3 results dynamically.

Readability and Maintainability:

Lateral joins can make the query easier to read and maintain if they are used correctly. This is particularly true for queries with multiple correlated subqueries or when there’s a natural separation of concerns between outer and inner logic.

#### Performance Considerations

Indexing Requirements:
With LEFT JOIN LATERAL, the derived table can be indexed independently, potentially allowing more efficient execution plans.
Window functions often rely on partitioning by customer IDs and ordering by dates within a single operation, which might not always lead to optimal indexing strategies.
Execution Plan:
The database optimizer may produce different execution plans for these two approaches. LEFT JOIN LATERAL can sometimes result in more efficient plans due to the ability to filter and limit rows earlier in the query.

#### Example
Query
SQL
SELECT
    customer_id,
    order_date,
    order_details
FROM customers c
LEFT JOIN LATERAL (
    SELECT order_date, order_details
    FROM orders o
    WHERE o.customer_id = c.customer_id
    ORDER BY order_date DESC
    LIMIT 3
) AS recent_orders ON true;
When Window Functions Outperform
Simplicity:
Window functions are more straightforward for simple ranking and ordering tasks.
Optimized Execution Plans:
In some cases, the database optimizer might produce an extremely efficient plan for window functions, particularly if there is a well-defined partitioning key (e.g., PARTITION BY customer_id).

#### Closing Summary

In summary, while both approaches can achieve similar results, lateral joins offer more flexibility and readability in complex query scenarios. However, they come with indexing requirements that need careful consideration. Window functions, on the other hand, are generally simpler to write and may outperform when dealing with straightforward ranking tasks.

#### Common Mistakes/Follow-Ups
Overly Complex Queries: Using lateral joins for very simple problems where window functions would suffice.
Incorrect Indexing: Failing to properly index the subquery in a LEFT JOIN LATERAL, which can degrade performance.
Execution Plan Optimization: Not leveraging database-specific features and optimizations for window functions.

**Comparison:**

```
FLOWCHART · MERMAID
gantt
  title Example Gantt Chart for Top-3 Recent Orders per Customer
  section Database Query Execution
  : LEFT JOIN LATERAL:
    2024-01-01,2024-01-05,done
  : Window Functions:
    2024-01-06,2024-01-10,done
```

**Code:**

```sql
-- Example query using LEFT JOIN LATERAL
SELECT
    customer_id,
    order_date,
    order_details
FROM customers c
LEFT JOIN LATERAL (
    SELECT order_date, order_details
    FROM orders o
    WHERE o.customer_id = c.customer_id
    ORDER BY order_date DESC
    LIMIT 3
) AS recent_orders ON true;

-- Example query using window functions
WITH ranked_orders AS (
    SELECT
        customer_id,
        order_date,
        order_details,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as rank
    FROM orders
)
SELECT
    customer_id,
    order_date,
    order_details
FROM customers c
JOIN ranked_orders ro ON c.customer_id = ro.customer_id AND ro.rank <= 3;
```

**Example input / output:**

```text
-- Input:
WITH ranked_orders AS (
    SELECT
        customer_id,
        order_date,
        order_details,
        ROW_NUMBER() OVER (PARTITION BY customer_id ORDER BY order_date DESC) as rank
    FROM orders
)
SELECT
    customer_id,
    order_date,
    order_details
FROM customers c
JOIN ranked_orders ro ON c.customer_id = ro.customer_id AND ro.rank <= 3;

-- Output:
+-------------+------------+-----------------+
| customer_id | order_date |     order_details|
+-------------+------------+-----------------+
|           1 | 2023-03-01 |    Order C       |
|           1 | 2023-02-01 |    Order B       |
|           1 | 2023-01-01 |    Order A       |
|           2 | 2023-06-01 |    Order E       |
|           2 | 2023-05-01 |    Order D       |
+-------------+------------+-----------------+
```

**Data:**

```json
{
  "customers": [
    { "customer_id": 1, "name": "Alice" },
    { "customer_id": 2, "name": "Bob" }
  ],
  "orders": [
    { "order_id": 1, "customer_id": 1, "order_date": "2023-01-01", "order_details": "Order A" },
    { "order_id": 2, "customer_id": 1, "order_date": "2023-02-01", "order_details": "Order B" },
    { "order_id": 3, "customer_id": 1, "order_date": "2023-03-01", "order_details": "Order C" },
    { "order_id": 4, "customer_id": 2, "order_date": "2023-05-01", "order_details": "Order D" },
    { "order_id": 5, "customer_id": 2, "order_date": "2023-06-01", "order_details": "Order E" }
  ]
}
```

**Flow:**

1. Identify Query Complexity
2. Use Lateral Join for ComplexLogic
3. Consider IndexingRequirements
4. Leverage Optimized ExecutionPlans
5. Simple Ranking Tasks
6. Use Window Functions Directly
7. Leverage Database-SpecificOptimizations

