---
type: "practice"
difficulty: "Deep"
---

# How do you decide between refreshing a materialized view ON COMMIT versus on a schedule, given the write amplification cost on the base table? Give a load scenario.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized View Refresh Correctness](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between refreshing a materialized view (MV) ON COMMIT versus on a scheduled basis in a database setting, the primary factors to consider are write amplification costs and their impact on both base table performance and overall system load. Write amplification is the phenomenon where multiple writes occur due to changes being propagated through the MVs, which can significantly degrade write performance if not managed properly.

To illustrate this with a load scenario, let’s assume we have an e-commerce application that performs frequent updates to product inventory tables. These updates could come from various sources such as user actions, automatic stock adjustments, or supplier notifications. Let's break down the considerations step by step:

Understanding Write Amplification: When changes are made to a base table, if those changes need to be propagated through multiple MVs ON COMMIT, it can lead to excessive write operations on the underlying storage system. This is particularly problematic in scenarios where there are high-frequency writes.

Scenario Setup:

Base Table: The product inventory table that gets updated frequently.
Materialized Views (MVs): Several MVs that aggregate different metrics such as total stock, low-stock alerts, and historical sales trends based on the base table.

ON COMMIT Refreshing:

#### Pros
Immediate consistency in all MVs. Any change to the base table is immediately reflected in the MVs.
Reduces the need for scheduled refreshes, potentially saving time and resources.

#### Cons
High write amplification if there are many MVs that depend on changes in the base table. Each commit could result in multiple writes to the MVs.
Increased load on the database as it has to handle both the updates to the base table and the additional writes required for MV refreshes.

Scheduled Refreshing:

#### Pros
Reduced write amplification because MVs are only refreshed at pre-defined intervals, reducing the number of write operations.
Can improve overall system performance by spreading out the load over time.

#### Cons
Potential inconsistency in MV data until the next scheduled refresh. This might not be acceptable if real-time or near-real-time analytics are required.

Load Scenario Analysis:

Assume our base table gets updated 10,000 times per minute due to various business activities.
Each update involves multiple MVs that need to be refreshed ON COMMIT, leading to significant write amplification (e.g., 30 writes per commit).
This means the database needs to handle an additional 3 million writes per minute due to MV refreshes, severely impacting performance and increasing storage costs.

Optimization Strategy:

Use a hybrid approach where critical MVs are refreshed ON COMMIT for immediate consistency, while less critical MVs can be scheduled at lower frequencies (e.g., hourly).
Implement caching mechanisms to reduce the load on base tables during high-traffic periods.

#### Cons
ider partitioning and indexing strategies to optimize write performance.

**Comparison:**

| Product ID | Quantity |
|------------|----------|
| 1          | 60       |
| 2          | 75       |
| Date       | Total Sales |
|------------|-------------|
| 2023-10-01 | 50          |

**Code:**

```sql
-- Example SQL for refreshing a materialized view ON COMMIT

CREATE MATERIALIZED VIEW mv_inventory AS
SELECT product_id, SUM(quantity) as total_stock
FROM inventory
GROUP BY product_id;

-- Enable ON COMMIT refresh
ALTER MATERIALIZED VIEW mv_inventory SET (ON_COMMIT_REFRESH = true);

-- Scheduling a MV to refresh on a specific interval
CREATE MATERIALIZED VIEW mv_sales_trends AS
SELECT DATE_TRUNC('day', sales_date) as day, SUM(quantity) as total_sales
FROM sales_data
GROUP BY day;

ALTER MATERIALIZED VIEW mv_sales_trends SET (REFRESH_INTERVAL => INTERVAL '1' HOUR);
```

**Example input / output:**

```text
Initial State:
- Inventory Table: [product_id=1, quantity=50], [product_id=2, quantity=75]
- MV Inventory: [product_id=1, total_stock=50], [product_id=2, total_stock=75]

After Commit (Immediate Refresh):
- Update Inventory: product_id=1, quantity=60
- New State of Inventory Table:
  - [product_id=1, quantity=60]
  - [product_id=2, quantity=75]
- Updated MV Inventory:
  - [product_id=1, total_stock=60]
  - [product_id=2, total_stock=75]

After Scheduled Refresh (Hourly):
- At the next scheduled time, update MV Sales Trends to reflect changes in sales_data.
```

**Data:**

```json
{
  "inventory": [
    { "product_id": 1, "quantity": 50 },
    { "product_id": 2, "quantity": 75 }
  ],
  "sales_data": [
    { "sales_date": "2023-10-01", "quantity": 20 },
    { "sales_date": "2023-10-01", "quantity": 30 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Base Table Update
4. Write to Base Table
5. Immediate Refresh?
6. MV Refreshed ON COMMIT
7. Scheduled MV Refresh
8. MV Updated Consistently withBase Table
9. MV Updated at ScheduledInterval

