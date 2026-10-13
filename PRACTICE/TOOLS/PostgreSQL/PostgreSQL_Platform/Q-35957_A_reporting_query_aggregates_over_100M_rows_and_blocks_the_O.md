---
type: "practice"
difficulty: "Deep"
---

# A reporting query aggregates over 100M rows and blocks the OLTP workload; how would you decide between a materialized view, a read replica, or offloading to a warehouse like Redshift/Snowflake?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Lag and Read Your Writes](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Replica_Lag_and_Read_Your_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To address the issue of a reporting query blocking OLTP workloads in PostgreSQL, we need to carefully consider trade-offs between materialized views, read replicas, and offloading to data warehouses like Redshift or Snowflake.

Triaging the Problem:

The core challenge is balancing the performance needs of real-time transactional operations (OLTP) with those of reporting queries. A blocking query can significantly degrade system responsiveness. We need to evaluate whether a materialized view, a read replica, or a full-scale data warehousing solution would be most effective.

#### Assumptions
The OLTP workload is write-heavy and time-sensitive.
Reporting queries are complex aggregates over large datasets.
We aim for minimal latency in both OLTP and reporting operations.

#### First-Principles Solution
1. Materialized View

A materialized view stores the result of a query as an actual table on disk, reducing the need to compute results at runtime. This can greatly improve read performance but comes with several caveats:

Update Latency: Materialized views require frequent updates which can impact OLTP performance.
Storage Overhead: Large amounts of data mean higher storage costs and potential I/O bottlenecks.
Complexity: Maintaining materialized views, especially with complex queries, requires careful management.
SQL
CREATE MATERIALIZED VIEW sales_summary AS
SELECT customer_id, SUM(sales) as total_sales
FROM orders
GROUP BY customer_id;
2. Read Replica

A read replica is a database that mirrors the primary database in near real-time. It can handle reporting queries without affecting the OLTP workload:

Performance: Read replicas are optimized for reads, providing faster query performance.
Latency: The lag between the primary and replica can be significant but manageable with proper tuning (e.g., asynchronous replication).
Scalability: Can distribute read load across multiple replicas.
SQL
CREATE REPLICATION SLAVE orders_replica FROM 'primary_host' PREFER REPLICA;
3. Data Warehouse (Redshift/Snowflake)

A data warehouse is designed for analytics and can handle complex queries over large datasets efficiently:

Performance: Optimized for ad-hoc queries, supporting advanced analytical functions.
Scalability: Can scale horizontally to handle massive amounts of data.
Cost: Higher cost due to storage and compute resources.
SQL
-- Example ETL process in Snowflake
CREATE OR REPLACE TABLE sales_summary AS
SELECT customer_id, SUM(sales) as total_sales
FROM orders
GROUP BY customer_id;
Decision Criteria:
Latency Requirements: If near real-time reporting is critical, a read replica or Redshift might be better.
Complexity and Maintenance: Materialized views require frequent updates but offer the simplest implementation for simple queries.
Cost Considerations: Offloading to a data warehouse can be cost-prohibitive unless strict budget constraints are relaxed.

#### Closing Summary

In summary, choosing between materialized views, read replicas, or offloading to a data warehouse depends on the specific needs and trade-offs. Materialized views offer simplicity but may not handle complex queries well. Read replicas provide near real-time reporting at the cost of some latency. Data warehouses excel in handling large-scale analytics but come with higher costs.

#### Common Mistakes
Overlooking the update overhead for materialized views.
Underestimating the storage and I/O requirements for materialized views.
Ignoring potential data consistency issues between primary and replicas.

**Comparison:**

| Customer ID | Total Sales |
|-------------|-------------|
| 123         | $50,000     |

**Code:**

```sql
-- Materialized View Example
CREATE MATERIALIZED VIEW sales_summary AS
SELECT customer_id, SUM(sales) as total_sales
FROM orders
GROUP BY customer_id;

-- Read Replica Example
CREATE REPLICATION SLAVE orders_replica FROM 'primary_host' PREFER REPLICA;

-- Data Warehouse ETL Example (Snowflake)
CREATE OR REPLACE TABLE sales_summary AS
SELECT customer_id, SUM(sales) as total_sales
FROM orders
GROUP BY customer_id;
```

**Example input / output:**

```
Input: SELECT SUM(sales) FROM orders WHERE date >= '2023-01-01' AND date <= '2023-06-30';
Output (Materialized View):
JSON
Copy
{
  "total_sales": 500000.00
}
Output (Read Replica):
JSON
Copy
{
  "total_sales": 499875.00
}
```

**Data:**

```json
{
  "customer_id": 123,
  "total_sales": 5000.98
}
```

**Flow:**

1. OLTP Workload
2. Materialized View
3. Update Latency
4. Storage Overhead
5. Read Replica
6. Latency
7. Scalability
8. Data Warehouse(Redshift/Snowflake)
9. Performance
10. Cost Considerations

