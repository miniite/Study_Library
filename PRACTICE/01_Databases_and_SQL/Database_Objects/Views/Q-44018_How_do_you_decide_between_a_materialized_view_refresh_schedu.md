---
type: "practice"
difficulty: "Deep"
---

# How do you decide between a materialized view refresh schedule and refreshing on demand?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Materialized View Trade-offs](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide between a materialized view (MV) refresh schedule and refreshing on-demand, it's crucial to understand both approaches deeply. A materialized view stores its results in a physical table, which can be refreshed periodically or on-demand. The choice depends on the trade-offs related to query performance, storage costs, maintenance overhead, and data freshness.

#### Triage

The question is essentially about balancing between real-time data availability and the cost of maintaining materialized views. We need to consider the following:

Query Performance: Materialized views can significantly reduce query response times by storing precomputed results.
Data Freshness: How often should we refresh the data to ensure it remains current?
Cost: Refreshing on-demand involves higher latency and potential system impacts, while scheduled refreshing consumes resources but provides predictable costs.
Step-by-Step Solution

Understand the Query Workload:

Evaluate the frequency of queries against the materialized view.
Assess how often the underlying data changes.

Data Freshness Requirements:

Determine if real-time updates are necessary or if a delay is acceptable.

#### Cons
ider the impact of delayed updates on downstream processes and user expectations.

Cost Analysis:

Calculate the cost implications of both approaches, including storage and computational resources.
Assess the frequency and duration of refresh operations for scheduled refreshing.

Maintenance Overhead:

Evaluate the complexity and maintenance overhead of managing schedules versus handling on-demand refreshes.

Performance Impact:

Analyze the performance gains from materialized views compared to real-time queries.

#### Cons
ider any potential bottlenecks during refresh operations.

#### Example Scenario

Suppose we have a large e-commerce platform with a Sales table and a view that calculates daily sales figures, which are used in multiple dashboards. The underlying data changes hourly due to new transactions.

Refresh Schedule: If the query workload is high but not extremely time-sensitive, we might opt for a scheduled refresh every 2 hours. This ensures relatively current data while minimizing resource consumption.

On-Demand Refresh: For critical queries that require the most recent data, refreshing on-demand could be beneficial. However, this would introduce higher latency and potential system load spikes.

#### Conclusion

In summary, materialized views offer a balance between query performance and data freshness but come with maintenance costs and overhead. The decision hinges on understanding the specific needs of your application and the trade-offs involved. A hybrid approach might also be viable, using both scheduled refreshes for most queries and on-demand refreshing for critical cases.

#### Common Mistakes
Overlooking Data Freshness Requirements: Assuming that all queries need real-time data without considering the impact.
Ignoring Maintenance Costs: Failing to account for the overhead of managing schedules or handling on-demand requests.
Not Considering Query Patterns: Making decisions based solely on theoretical considerations rather than empirical evidence.

**Comparison:**

| Day        | Total_Sales |
|------------|-------------|
| 2023-09-01 | 1050.0      |
| 2023-09-02 | 840.0       |
| ...        | ...         |

**Code:**

```sql
-- Example SQL for a materialized view with scheduled refresh
CREATE MATERIALIZED VIEW daily_sales AS
SELECT date_trunc('day', sale_time) as day, SUM(amount) as total_sales
FROM sales
GROUP BY date_trunc('day', sale_time);

-- Refresh on schedule (e.g., every 2 hours)
REFRESH MATERIALIZED VIEW CONCURRENTLY daily_sales;

-- On-demand refresh
REFRESH MATERIALIZED VIEW daily_sales;
```

**Example input / output:**

```text
Input:
SELECT SUM(total_sales)
FROM daily_sales
WHERE day >= CURRENT_DATE - INTERVAL '7 days';

Output:
3200.0 (Assuming the sum of total sales over the last 7 days)
```

**Data:**

```json
{
  "sales": [
    { "sale_id": 1, "amount": 50.0, "sale_time": "2023-09-01T14:00:00Z" },
    { "sale_id": 2, "amount": 75.0, "sale_time": "2023-09-01T15:00:00Z" },
    // ... more sales
  ]
}
```

**Flow:**

1. Yes
2. No
3. Query against MaterializedView
4. Check if Data Freshness meetsRequirements
5. Data Freshness Required?
6. Schedule Refreshes Based onWorkload
7. Perform On-Demand Refreshesas Needed
8. Maintain Schedule andMonitor Performance
9. Maintain Flexibility for CriticalQueries

