---
type: "practice"
difficulty: "Intermediate"
---

# How would you group orders by both region and month at the same time to build a summary report?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ GROUP BY vs PARTITION BY](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/GROUP_BY_vs_PARTITION_BY.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To group orders by both region and month, we need to aggregate our data in such a way that it provides us with the summarized information based on these two dimensions. The key idea is to use SQL's GROUP BY clause effectively alongside aggregation functions like SUM, COUNT, or AVG. This process involves selecting relevant columns, applying the grouping logic, and then performing any necessary aggregations.

**Code:**

```sql
-- Sample SQL query to group orders by region and month

SELECT
    region,
    DATE_FORMAT(order_date, '%Y-%m') AS order_month,
    COUNT(*) AS total_orders,
    SUM(total_amount) AS total_revenue
FROM
    orders
GROUP BY
    region,
    order_month;
```

**Example input / output:**

```

Here’s a worked example with sample input data and the expected output:

Sample Input Data (orders table):

JSON
Copy
[
  {"order_id": 1, "region": "East", "order_date": "2023-01-15", "total_amount": 100},
  {"order_id": 2, "region": "West", "order_date": "2023-01-20", "total_amount": 200},
  {"order_id": 3, "region": "East", "order_date": "2023-02-18", "total_amount": 150},
  {"order_id": 4, "region": "West", "order_date": "2023-02-22", "total_amount": 250}
]

Expected Output:

JSON
Copy
[
  {"region": "East", "order_month": "2023-01", "total_orders": 1, "total_revenue": 100},
  {"region": "West", "order_month": "2023-01", "total_orders": 1, "total_revenue": 200},
  {"region": "East", "order_month": "2023-02", "total_orders": 1, "total_revenue": 150},
  {"region": "West", "order_month": "2023-02", "total_orders": 1, "total_revenue": 250}
]
```

