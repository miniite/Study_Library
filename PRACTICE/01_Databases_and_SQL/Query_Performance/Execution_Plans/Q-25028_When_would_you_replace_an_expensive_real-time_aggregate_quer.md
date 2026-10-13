---
type: "practice"
difficulty: "Deep"
---

# When would you replace an expensive real-time aggregate query with a materialized view and how do you handle refresh?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Statistics and Sargability](<../../../../01_Databases_and_SQL/Query_Performance/Execution_Plans/Statistics_and_Sargability.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address when to replace an expensive real-time aggregate query with a materialized view, it's crucial to understand that both techniques serve different purposes in optimizing database performance. Real-time queries provide up-to-the-minute data but can be resource-intensive and slow down your database operations. On the other hand, materialized views store pre-computed results of complex queries, reducing the need for real-time computation at the cost of some freshness.

When deciding to switch from a real-time aggregate query to a materialized view, consider the following:

Query Frequency and Performance Impact: If your application performs this query frequently (e.g., more than once per minute), or if it significantly impacts database performance during peak times, you might benefit from caching results in a materialized view.

Data Freshness vs. Computation Cost: Materialized views can introduce a slight delay between the actual data changes and when those changes are reflected in your reports or dashboards. If this delay is acceptable for your use case, then materialized views can be an effective choice.

Storage Requirements: Evaluate whether storing pre-computed results will fit within available storage constraints. This includes both disk space and memory usage if the view is being accessed frequently.

Maintenance Overhead: Consider the cost of maintaining a materialized view, which involves ensuring it’s updated correctly when underlying data changes. You’ll need to set up triggers or use other mechanisms to keep the view in sync with your data.

To handle the refresh of a materialized view:

Synchronous Refreshes: This approach is straightforward and ensures that the view reflects the most recent state but can cause locking issues if not done carefully.
Asynchronous Refreshes (Cron Jobs or Scheduled Tasks): This approach keeps the main database operations flowing smoothly while periodically updating the view, which might be more suitable for less frequently changing data.

Let's break down a step-by-step process to implement this:

Identify and Profile the Real-Time Query:

Examine the SQL query that you are currently running in real-time.
Use database tools like EXPLAIN PLAN or equivalent features to understand its execution path, cost, and performance bottlenecks.

Design the Materialized View Schema:

Determine which columns should be included in the view for optimal performance.
Decide on a suitable partitioning strategy if your data is large (e.g., time-based partitions).

Implement Triggers or Scheduling for Refreshes:

For synchronous refreshes, you might need to add triggers that update the materialized view whenever there are changes in underlying tables.
For asynchronous refreshes, create a scheduled job using your database’s scheduler (e.g., cron jobs in PostgreSQL).

Monitor Performance and Adjust as Needed:

Continuously monitor the performance of both the real-time query and the materialized view to ensure they meet your requirements.
Optimize further by tweaking indexes, adjusting partitioning strategies, or even switching back to a real-time solution if necessary.

**Comparison:**

| Day       | Total Sales |
|-----------|-------------|
| 2023-10-05|     100      |
| 2023-10-06|     150      |

**Code:**

```sql
-- Example SQL for creating a materialized view in PostgreSQL

CREATE MATERIALIZED VIEW sales_summary AS
SELECT date_trunc('day', order_time) as day, SUM(order_amount) as total_sales
FROM orders
GROUP BY date_trunc('day', order_time);

-- Synchronous refresh example with a trigger
CREATE OR REPLACE FUNCTION update_materialized_view()
RETURNS TRIGGER AS $$
BEGIN
    REFRESH MATERIALIZED VIEW sales_summary;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER sync_sales_summary AFTER INSERT ON orders
FOR EACH ROW EXECUTE FUNCTION update_materialized_view();

-- Asynchronous refresh example using a scheduled task in PostgreSQL
SELECT pg_notify('refresh_sales_summary', '');
```

**Example input / output:**

```text
Input:
- Real-time query: SELECT SUM(order_amount) FROM orders;

Output:
- Materialized view: sales_summary, with results pre-computed daily.

Dry-run Example (Sync Refresh):
- Trigger fires after new order is inserted.
- `REFRESH MATERIALIZED VIEW sales_summary;` executes to update the materialized view.
```

**Data:**

```json
{
  "orders": [
    {"order_id": 1, "order_time": "2023-10-05T14:30:00Z", "order_amount": 100},
    {"order_id": 2, "order_time": "2023-10-06T09:00:00Z", "order_amount": 150}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Real-time Query
4. Is Query Frequent?
5. Migrate to Materialized View
6. Keep Real-time Query
7. Design MV Schema
8. ImplementSynchronous/AsynchronousRefreshes
9. Monitor Performance

