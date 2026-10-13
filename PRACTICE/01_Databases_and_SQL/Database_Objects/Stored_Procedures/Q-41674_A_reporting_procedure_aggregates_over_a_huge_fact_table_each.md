---
type: "practice"
difficulty: "Intermediate"
---

# A reporting procedure aggregates over a huge fact table each time. How would you use materialized views or summary tables to speed it up?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Stored Procedure Performance and Design](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize a reporting procedure that aggregates over a large fact table, we can leverage materialized views or summary tables. The core idea is to precompute and store the aggregated results in these structures, thereby avoiding repeated heavy computations each time the report runs.

#### Triaging the Question

The interviewer is probing how you would improve the performance of an aggregation operation that currently runs over a large fact table. Materialized views and summary tables are both techniques used to store precomputed data, allowing for faster query execution at the cost of additional storage overhead. We need to understand the trade-offs between these two approaches and decide which one is more appropriate.

#### Assumptions
The fact table contains millions or billions of records.
The aggregation operation involves complex calculations (e.g., SUM, AVG, COUNT) over multiple columns.
The reporting procedure runs frequently but not in real-time.

#### First-Principles Solution
Step 1: Understanding Materialized Views

Materialized views are database objects that store the results of a SELECT query. They can significantly speed up queries by providing precomputed data. However, materialized views require periodic refreshes to keep their contents current with the underlying tables. This involves additional administrative overhead and storage costs.

Step 2: Understanding Summary Tables

Summary tables are similar to materialized views but typically involve custom logic for computing aggregates. They can be more flexible than materialized views because they allow for complex aggregation operations that might not fit into a simple SQL query. However, summary tables also require periodic updates and maintenance.

Step 3: Choosing Between Materialized Views and Summary Tables

To decide which approach to use, we need to consider the following factors:

Frequency of Data Changes: If data changes frequently, materialized views may be easier to maintain as they can be automatically refreshed.
Complexity of Aggregation Operations: If the aggregation operations are complex, summary tables might offer more flexibility and performance benefits.
Storage Constraints: Materialized views consume additional storage space, which must be considered in terms of database capacity.
Step 4: Implementation Steps
Design the Aggregation Logic: Break down the aggregation into simpler steps if necessary.
Create a Summary Table or Materialized View:
SQL
CREATE TABLE summary_table AS
SELECT column1, SUM(column2) as total_sum FROM fact_table GROUP BY column1;
Schedule Refreshes: If using materialized views, set up a job to refresh the view at regular intervals.
Monitor Performance: Continuously monitor the performance impact and make adjustments as needed.
Step 5: Monitoring and Maintenance
Refresh Scheduling: Ensure that the summary table or materialized view is refreshed frequently enough to avoid staleness but not too often to avoid unnecessary overhead.
Storage Management: Regularly review storage usage to ensure it remains within acceptable limits.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By using materialized views or summary tables, we can significantly reduce the execution time of our aggregation operation. However, this comes with additional maintenance and storage costs. It's crucial to carefully weigh these trade-offs based on the specific requirements of your application.

#### Common mistakes
include:

Overlooking the need for periodic refreshes.
Not considering the complexity and flexibility needed in aggregation operations.
Failing to monitor performance and adjust strategies as data patterns evolve.

#### Follow-ups
might involve discussing how to handle real-time updates, partitioning large tables, or integrating these solutions with other database optimization techniques like indexing.

**Comparison:**

```
Create Summary Table
Select Aggregated Data
Refresh Materialized View
Query PerformanceImprovement
Maintenance and Monitoring
```

**Code:**

```sql
-- Example of creating a summary table
CREATE TABLE summary_table AS
SELECT category_id, SUM(amount) as total_sales FROM sales_fact GROUP BY category_id;

-- Example of refreshing a materialized view (assuming SQL Server)
ALTER INDEX ALL ON sales_materialized_view REBUILD;
```

**Example input / output:**

```text
Input:
SELECT SUM(amount) FROM sales_fact;

Output:
Total Sales: 900

Input for Summary Table:
CREATE TABLE summary_table AS
SELECT category_id, SUM(amount) as total_sales FROM sales_fact GROUP BY category_id;

Output for Summary Table:
Category ID | Total Sales
------------|------------
1           | 500
2           | 400
```

**Data:**

```json
{
  "fact_table": {
    "columns": ["category_id", "amount"],
    "rows": [
      {"category_id": 1, "amount": 200},
      {"category_id": 1, "amount": 300},
      {"category_id": 2, "amount": 400}
    ]
  },
  "summary_table": {
    "columns": ["category_id", "total_sales"],
    "rows": [
      {"category_id": 1, "total_sales": 500},
      {"category_id": 2, "total_sales": 400}
    ]
  }
}
```

**Flow:**

1. +-----------------------+
2. | Create Summary Table  |
3. +----------+-----------+
4. |           v
5. +----------v------------+
6. | Select Aggregated Data|
7. +----+------+---------+
8. |          |
9. v          v
10. +----------v--------+ +---------+
11. | Query Performance | | Maintenance|
12. +--------------------+ +-------------+

