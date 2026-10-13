---
type: "practice"
difficulty: "Deep"
---

# A materialized view on Snowflake or BigQuery has limits on the SQL it supports; how do you decide when to use a native materialized view versus a scheduled table build?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized View Trade-offs](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between using a native materialized view in Snowflake or BigQuery and building a scheduled table, we need to consider several factors related to performance, cost, data freshness, and SQL support. Native materialized views can provide significant query performance benefits but may have limitations on the types of queries they can handle. Let's break down the decision-making process from first principles.

**Comparison:**

| Column Name    | Data Type   |
|----------------|-------------|
| customer_id    | INTEGER     |
| total_sales    | FLOAT       |

**Code:**

```
Example of a simple view creation in Snowflake

CREATE OR REPLACE VIEW sales_view AS SELECT customer_id, SUM(sales_amount) as total_sales FROM sales_table GROUP BY customer_id;

CODE
Copy

```sql
-- Example of a scheduled table build in BigQuery
CREATE TABLE my_dataset.my_materialized_view AS
SELECT customer_id, SUM(sales_amount) as total_sales
FROM `project-id.dataset.sales_table`
GROUP BY customer_id;
```

**Example input / output:**

```text
-- Example input query using the materialized view in Snowflake
SELECT customer_id, total_sales
FROM sales_view;

-- Expected output:
+-----------+-------------+
| customer_id | total_sales |
+-----------+-------------+
| 123        | 5000.00     |
| 456        | 7500.00     |
+-----------+-------------+
```

**Data:**

```json
{
  "sales_table": {
    "columns": ["customer_id", "transaction_date", "sales_amount"],
    "schema": [
      {"name": "customer_id", "type": "INTEGER"},
      {"name": "transaction_date", "type": "DATE"},
      {"name": "sales_amount", "type": "FLOAT"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Identify Query Requirements
8. Are the queries simpleaggregations?
9. Use Native Materialized View
10. Check for Complex Queriesand Joins
11. Does the query involvecomplex joins or subqueries?
12. Scheduled Table Build
13. Use Native Materialized View
14. Consider Data FreshnessRequirements
15. Is real-time data freshnesscritical?
16. Use Scheduled Table Build withNear-Real-Time Updates
17. Use Native Materialized Viewfor Performance

