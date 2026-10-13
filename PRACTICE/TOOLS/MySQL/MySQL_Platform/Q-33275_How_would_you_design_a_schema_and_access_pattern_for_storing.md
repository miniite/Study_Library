---
type: "practice"
difficulty: "Deep"
---

# How would you design a schema and access pattern for storing per-request API metrics (latency histograms) in MySQL cheaply, and when do you push it to a TSDB instead?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Activity Timeline Schema](<../../../../TOOLS/MySQL/MySQL_Platform/Activity_Timeline_Schema.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store per-request API metrics like latency histograms efficiently in MySQL while keeping costs low, we need to carefully design both our schema and access patterns. Typically, these metrics require frequent updates but infrequent queries over long periods. We'll consider a hybrid storage strategy that uses MySQL for immediate data capture and cost-effective caching, and a time-series database (TSDB) like InfluxDB or Prometheus for long-term storage and analysis.

#### Triaging the Question

The question is probing how to balance real-time metrics collection with historical analysis in a cost-efficient manner. We need to understand the trade-offs between MySQL's ACID properties and write performance, versus TSDBs' optimized designs for time-series data. Key assumptions include:

Frequent writes (latency histograms) but infrequent reads.
Historical retention needs are significant (months or years).
Cost constraints require minimizing storage overhead.

#### First-Principles Solution
MySQL Schema Design:
Use an InnoDB engine for its transactional capabilities and row-level locking.
Create a table per API endpoint to simplify queries and reduce contention.
Access Patterns:
Insert new latency data using batch operations or multi-row inserts to minimize the number of writes.
Cost Considerations:
MySQL is not optimized for high write throughput, so it may become a bottleneck.
Threshold for TSDB Migration:
When historical retention exceeds 6 months and write frequency decreases significantly.
Step-by-Step Solution

MySQL Schema Design:

SQL
CREATE TABLE api_latency (
    request_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    endpoint VARCHAR(255) NOT NULL,
    timestamp DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    latency DECIMAL(10, 4) NOT NULL
);
request_id: Unique identifier for each request.
endpoint: API endpoint being monitored.
timestamp: Time of the metric collection.

Access Patterns:

Use batch inserts to reduce write overhead:
SQL
INSERT INTO api_latency (endpoint, timestamp, latency)
VALUES ('/api/v1/data', NOW(), 50),
       ('/api/v1/data', NOW(), 60);

Cost Considerations:

Monitor MySQL's write performance and storage costs.
Optimize by setting up indexes on endpoint and timestamp.

Threshold for TSDB Migration:

When the retention period exceeds 6 months, consider moving old data to a TSDB.

#### Closing Summary

By using MySQL for real-time metric collection with optimized batch writes and indexing, we can keep costs low initially. However, as historical data accumulates, the load on MySQL may become unsustainable. At that point, migrating older metrics to a TSDB is advisable to balance performance and cost effectively.

#### Common Mistakes/Follow-Ups
Failing to monitor write performance: Ignoring this could lead to unexpected costs or system degradation.
Overlooking long-term retention needs: Ensuring data is moved before it becomes too costly in MySQL.
Not considering alternative TSDB options: Evaluating other solutions like Prometheus for specific use cases.

**Code:**

```sql
CREATE TABLE api_latency (
    request_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    endpoint VARCHAR(255) NOT NULL,
    timestamp DATETIME NOT NULL DEFAULT CURRENT_TIMESTAMP,
    latency DECIMAL(10, 4) NOT NULL
);
```

