---
type: "practice"
difficulty: "Deep"
---

# How would you design a materialized view for a heavy dashboard query, keep it fresh with REFRESH CONCURRENTLY, and surface the staleness/consistency trade-off to product?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a materialized view (MV) for a heavy dashboard query, we need to ensure that the MV remains up-to-date while minimizing the impact on system performance and resources. The REFRESH CONCURRENTLY command is key here as it allows us to update the MV concurrently with other operations without blocking them. However, there’s a trade-off between freshness of data and resource usage during refreshes. Let's break this down step-by-step.

#### Triaging the Question

The question asks about designing an MV for a heavy dashboard query, keeping it fresh with REFRESH CONCURRENTLY, and explaining the staleness/consistency trade-offs to product stakeholders. This involves understanding how MVs work, the impact of concurrent refreshes, and communicating these details effectively.

Core Solution Overview

We will design an efficient MV that balances freshness and resource usage by leveraging REFRESH CONCURRENTLY. We'll also monitor and communicate the potential delays in data staleness to ensure product stakeholders are informed about the trade-offs involved.

Detailed First-Principles Solution
Identify the Heavy Query: Analyze the dashboard query to understand its complexity, what joins and aggregations it performs, and how often it's executed.
Design the MV:
Choose an appropriate name for the MV that reflects its purpose (e.g., dashboard_sales_summary).
Ensure the MV’s schema matches the output of the heavy query to avoid data mismatches during refreshes.
Implement Refresh Strategy with Concurrently:
Use CREATE MATERIALIZED VIEW and REFRESH CONCURRENTLY to ensure that the MV can be updated while other operations are ongoing without blocking them.
Monitor Performance and Staleness:
Set up monitoring for refresh times and data freshness.
Alert stakeholders about potential delays due to concurrent refreshes.
Step-by-Step Solution

Analyze the Query:

SQL
SELECT customer_id, SUM(sales_amount) AS total_sales
FROM sales
JOIN customers ON sales.customer_id = customers.id
WHERE sale_date >= '2023-01-01'
GROUP BY customer_id;

Design the MV:

SQL
CREATE MATERIALIZED VIEW dashboard_sales_summary AS
SELECT customer_id, SUM(sales_amount) AS total_sales
FROM sales
JOIN customers ON sales.customer_id = customers.id
WHERE sale_date >= '2023-01-01'
GROUP BY customer_id;

Concurrent Refresh:

SQL
REFRESH MATERIALIZED VIEW CONCURRENTLY dashboard_sales_summary;

Monitoring and Communication:

Use pg_stat_activity to monitor refresh times.
Set up alerts for when the refresh time exceeds a certain threshold.

#### Closing Summary
+ Common Mistakes/Follow-Ups
The primary goal is to ensure that the MV remains fresh while minimizing impact on system performance during refreshes.
The trade-off involves balancing freshness against potential delays in data staleness. Communicate this effectively to product stakeholders, highlighting when and how often updates might lag.

#### Common Mistakes
Overlooking Indexes: Ensure proper indexes are created on the MV to speed up queries and refresh operations.
Ignoring Performance Impact: Failing to monitor and manage the performance impact of concurrent refreshes can lead to unexpected resource contention.
Miscommunication: Not clearly explaining the trade-offs in data freshness to stakeholders, which can cause confusion or misalignment.
Additional Considerations
Evaluate whether a more complex approach like materialized views with dynamic updates might be necessary if the query is extremely heavy and frequent.

#### Cons
ider using PARTITION BY clauses if applicable to further optimize the MV.

**Comparison:**

```
tableChart
    row "customer_id" "total_sales"
    row "101" "500"
    row "102" "600"
```

**Code:**

```sql
CREATE MATERIALIZED VIEW dashboard_sales_summary AS
SELECT customer_id, SUM(sales_amount) AS total_sales
FROM sales
JOIN customers ON sales.customer_id = customers.id
WHERE sale_date >= '2023-01-01'
GROUP BY customer_id;

REFRESH MATERIALIZED VIEW CONCURRENTLY dashboard_sales_summary;
```

**Example input / output:**

| Input Query             | Output MV Data            |
|-------------------------|--------------------------|
| `SELECT * FROM dashboard_sales_summary;` |
| - `customer_id`: 101, `total_sales`: 500 <br> - `customer_id`: 102, `total_sales`: 600 |

**Data:**

```json
{
  "sales": [
    {"id": 1, "customer_id": 101, "sale_date": "2023-04-01", "sales_amount": 500},
    {"id": 2, "customer_id": 102, "sale_date": "2023-04-02", "sales_amount": 600},
    // More sales records
  ],
  "customers": [
    {"id": 101, "name": "Alice"},
    {"id": 102, "name": "Bob"},
    // More customer records
  ]
}
```

**Flow:**

1. Yes
2. No
3. Init MV
4. Is Refresh Needed?
5. REFRESH MATERIALIZED VIEWCONCURRENTLY
6. MV Fresh
7. Wait for Next Refresh

