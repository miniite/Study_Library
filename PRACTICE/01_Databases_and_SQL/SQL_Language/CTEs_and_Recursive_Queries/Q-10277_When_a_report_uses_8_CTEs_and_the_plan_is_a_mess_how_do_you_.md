---
type: "practice"
difficulty: "Deep"
---

# When a report uses 8 CTEs and the plan is a mess, how do you decide between splitting into temp tables versus keeping one statement?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Recursive CTEs and Materialization](<../../../../01_Databases_and_SQL/SQL_Language/CTEs_and_Recursive_Queries/Recursive_CTEs_and_Materialization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the complexity of handling multiple Common Table Expressions (CTEs) in a report, we need to balance clarity and performance. When 8 CTEs lead to an overly complex plan, splitting them into temporary tables can simplify query readability while maintaining performance. Here's how you decide between using temp tables or keeping one statement:

TL;DR
Temp Tables: Simplify the code by breaking down complex logic.
One Statement with CTEs: Maintain a single logical flow and potentially better optimizer performance.

#### Triage
Understanding the Question

The core of this question is about deciding between using Common Table Expressions (CTEs) or temporary tables to handle multiple queries in a report. The interviewer might be probing your understanding of query optimization, performance trade-offs, and database design principles.

#### Assumptions
Query Complexity: 8 CTEs indicate a complex logical flow.
Performance Concerns: The current plan is messy and potentially slow.
Readability and Maintainability: Code clarity is important for future maintenance.

#### First-Principles Solution

#### Performance Considerations

First, we need to understand the performance implications of both approaches:

CTEs:

#### Pros
CTEs allow you to break down complex logic into smaller, more understandable parts.

#### Cons
The optimizer may struggle with recursive and deeply nested queries, leading to a less efficient execution plan.

Temp Tables:

#### Pros
Temp tables can be used for caching intermediate results, which can significantly improve performance.

#### Cons
They introduce additional temporary storage and management overhead, which could impact performance if not managed properly.
Step-by-Step Solution

Analyze the Current Plan:

Review the current execution plan to identify bottlenecks.
Check if the query is using a recursive CTE or complex logic that might be causing inefficiency.

Identify Redundant Logic:

Look for repeated calculations or unnecessary intermediate results.
Use EXPLAIN or equivalent tools to understand how the database is executing the queries.

Evaluate Temp Table Benefits:

Create temporary tables for each significant CTE and insert data from those CTEs into the temp tables.
Join these temp tables in a final query, which can often lead to simpler execution plans.

Optimize Each Step:

For each step, consider if caching intermediate results would help.
Test different approaches with EXPLAIN to see how they perform.

Balance Complexity and Readability:

If the code becomes too complex even after splitting into temp tables, reconsider the logic.
Ensure that the split maintains readability while improving performance.

#### Closing Summary

In summary, when dealing with a report using 8 CTEs that results in an overly complex execution plan, you should consider breaking down the queries into temporary tables to simplify and potentially optimize performance. However, this decision should be made based on a thorough analysis of the current query, understanding the trade-offs between readability and performance.

#### Common Mistakes/Follow-Ups
Over-Simplification: Splitting too many steps may lead to overly complex temp table management.
Under-Simplification: Keeping everything in one statement might obscure logic but could perform better if the optimizer can handle it efficiently.
Performance Testing: Always test different approaches with real data before making a decision.

Would you like to explore any specific scenarios or further details about this approach?

**Comparison:**

```
CTE	Description
cte1	Orders before 2023-01-01
cte2	Products with the name 'Widget'
...	...
cte8	Customers from USA
Temp Table	Description
temp_cte1	Orders before 2023-01-01
temp_cte2	Products with the name 'Widget'
...	...
temp_cte8	Customers from USA
```

**Code:**

```sql
-- Example of splitting CTEs into temp tables

WITH cte1 AS (
    SELECT ...
),
cte2 AS (
    SELECT ...
),
...
cte8 AS (
    SELECT ...
)

SELECT * FROM cte1, cte2, ..., cte8 WHERE ...

-- Splitting into temp tables
CREATE TEMPORARY TABLE temp_cte1 AS (SELECT ...);
CREATE TEMPORARY TABLE temp_cte2 AS (SELECT ...);
...

SELECT * FROM temp_cte1, temp_cte2, ..., temp_cte8 WHERE ...
```

**Example input / output:**

```text
-- Input (CTEs)
WITH cte1 AS (
    SELECT * FROM orders WHERE order_date < '2023-01-01'
),
cte2 AS (
    SELECT * FROM products WHERE product_name = 'Widget'
),
...
cte8 AS (
    SELECT * FROM customers WHERE country = 'USA'
)

SELECT * FROM cte1, cte2, ..., cte8;

-- Output (Temp Tables)
CREATE TEMPORARY TABLE temp_cte1 AS (
    SELECT * FROM orders WHERE order_date < '2023-01-01'
);
CREATE TEMPORARY TABLE temp_cte2 AS (
    SELECT * FROM products WHERE product_name = 'Widget'
);
...

SELECT * FROM temp_cte1, temp_cte2, ..., temp_cte8;
```

**Data:**

```json
{
  "data": [
    {
      "cte1": [
        {"id": 1, "value": 10},
        {"id": 2, "value": 20}
      ],
      "cte2": [
        {"id": 3, "value": 30},
        {"id": 4, "value": 40}
      ]
    }
  ]
}
```

**Flow:**

1. (Root)
2. Child A
3. Child B
4. Sub-Child A
5. Leaf A

