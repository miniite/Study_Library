---
type: "practice"
difficulty: "Deep"
---

# When would you use a materialized view and how do you keep it fresh?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Materialized CTEs](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_CTEs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Materialized views in PostgreSQL are useful when you need to precompute and store query results that don't change often, but might be queried frequently. This approach helps reduce the load on the database by caching the results of complex queries, making it faster to retrieve them. To keep a materialized view fresh, we use triggers or functions to ensure that it reflects any changes in the underlying data.

Let's break down this concept from first principles and explore how you would implement such a solution.

**Comparison:**

```
product_id	total_sales
1	370
2	600
```

**Code:**

```sql
-- Example of creating a materialized view

CREATE MATERIALIZED VIEW sales_summary AS
SELECT product_id, SUM(sales_amount) as total_sales
FROM sales_data
GROUP BY product_id;

-- Refreshing the materialized view
REFRESH MATERIALIZED VIEW sales_summary;
```

**Example input / output:**

```text
-- Initial state of the materialized view creation and refresh

CREATE MATERIALIZED VIEW sales_summary AS
SELECT product_id, SUM(sales_amount) as total_sales
FROM sales_data
GROUP BY product_id;

REFRESH MATERIALIZED VIEW sales_summary;

-- After new data is added to the sales_data table

INSERT INTO sales_data (product_id, date, sales_amount)
VALUES (1, '2023-01-03', 120), (2, '2023-01-04', 220);

REFRESH MATERIALIZED VIEW sales_summary;

-- Result of the refreshed materialized view

SELECT * FROM sales_summary;
```

**Data:**

```json
{
  "sales_data": [
    {"product_id": 1, "date": "2023-01-01", "sales_amount": 100},
    {"product_id": 1, "date": "2023-01-02", "sales_amount": 150},
    {"product_id": 2, "date": "2023-01-01", "sales_amount": 200},
    {"product_id": 2, "date": "2023-01-02", "sales_amount": 180}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Detect Changes in Sales Data
4. Invoke REFRESHMATERIALIZED VIEW
5. Are there new sales?
6. Materialized View is Fresh
7. (Use Materialized View)

