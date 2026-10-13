---
type: "practice"
difficulty: "Basic"
---

# What is a materialized view in the warehouse and how could you use one to serve a stable, cheap feature to your model?

**Difficulty:** Basic · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

A materialized view in a data warehouse is a precomputed result of a query that is stored as a virtual table. This approach enables rapid access and reduces computation time by serving up-to-date, cached results without re-running the underlying queries. In an ML context, materialized views can be used to create stable feature sets that are readily available for model training and inference, ensuring consistency and reducing computational overhead.

#### Triaging the Question

The question is probing your understanding of data warehousing concepts and how they can be applied in a machine learning (ML) pipeline. Specifically, it asks you to explain materialized views and demonstrate their utility in serving stable features efficiently.

#### First-Principles Solution
What are Materialized Views?

Materialized views store the results of complex queries as tables, which can then be queried just like regular tables. These views update automatically when underlying data changes, ensuring that they always reflect current data. In a warehouse context, this means precomputing and caching query results to speed up read operations.

Why Use Materialized Views?
Performance: Reduces the need for complex queries at runtime by serving cached results.

#### Cons
istency: Ensures that all downstream systems access the latest computed data without re-executing expensive computations.
Cost Efficiency: Minimizes computational overhead, especially for read-heavy workloads.

#### Example
Use Case

Imagine you have a feature engineering process where certain features are derived from complex aggregations and transformations of raw data. These features need to be updated frequently but used consistently across multiple models or teams. By creating a materialized view that stores these precomputed features, you can:

Create the Materialized View: Define a SQL query that computes your desired features.

SQL
CREATE MATERIALIZED VIEW customer_features AS
SELECT
    customer_id,
    SUM(purchase_amount) OVER (PARTITION BY customer_id ORDER BY purchase_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as lifetime_spend,
    AVG(product_category) OVER (PARTITION BY customer_id) as avg_product_category
FROM purchases;

Update the View: Ensure that this view automatically updates when new data is inserted or existing data changes.

Query the View: Direct your ML models to use the materialized view for feature selection.

#### Closing Summary
and Common Mistakes/Follow-Ups

Using materialized views effectively requires balancing the cost of maintaining up-to-date results with the benefits of reduced computation at query time. Common mistakes include not setting proper refresh policies, leading to stale data, or overusing materialized views when simple caching could suffice. Follow-up questions might explore specific implementation details in your organization's warehouse system or trade-offs between different storage mechanisms.

**Comparison:**

| customer_id | lifetime_spend | avg_product_category |
|------------|---------------|---------------------|
| 1          | 500           | Electronics         |
| 2          | 300           | Books               |

**Code:**

```sql
CREATE MATERIALIZED VIEW customer_features AS
SELECT
    customer_id,
    SUM(purchase_amount) OVER (PARTITION BY customer_id ORDER BY purchase_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) as lifetime_spend,
    AVG(product_category) OVER (PARTITION BY customer_id) as avg_product_category
FROM purchases;
```

**Example input / output:**

```text
Input:
SELECT * FROM customer_features WHERE customer_id = 1;

Output:
+-------------+-------------------+-----------------+
| customer_id | lifetime_spend    | avg_product_category |
+-------------+-------------------+-----------------+
|           1 |                500 | Electronics      |
+-------------+-------------------+-----------------+
```

**Data:**

```json
{
  "customer_features": [
    {"customer_id": 1, "lifetime_spend": 500, "avg_product_category": "Electronics"},
    {"customer_id": 2, "lifetime_spend": 300, "avg_product_category": "Books"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Arrival
4. Refresh Materialized View?
5. Maintain View
6. Use Cached Results
7. Query Cache
8. Query Cache

