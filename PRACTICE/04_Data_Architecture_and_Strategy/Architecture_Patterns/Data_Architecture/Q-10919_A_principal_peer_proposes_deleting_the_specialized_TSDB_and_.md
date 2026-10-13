---
type: "practice"
difficulty: "Expert"
---

# A principal peer proposes deleting the specialized TSDB and OLAP stores and running all AIOps queries directly on the lake with Trino. Stress-test that: where does 'just query the lake' fall over for alerting query concurrency, freshness, and sub-second dashboards?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the proposal of running all AIOps queries directly on a data lake using Trino instead of specialized Time Series Database (TSDB) and Online Analytical Processing (OLAP) stores, we need to stress-test this approach against key performance criteria such as alerting query concurrency, freshness, and sub-second dashboards. The core issue is whether the lake's architecture can handle these specific use cases without degradation in performance or quality.

#### Triage

The principal peer proposes consolidating all AIOps queries onto a data lake using Trino, which is designed for handling complex analytics over large-scale datasets. We need to evaluate if this consolidation will meet the requirements of alerting query concurrency (i.e., can it handle multiple simultaneous high-frequency queries), freshness (i.e., how quickly can we get updated insights from the data), and sub-second dashboards (i.e., real-time visualization).

#### Assumptions
The lake contains the necessary data.
Trino is configured correctly for performance optimization.
The alerting system and dashboards are designed to be flexible enough to handle varying query loads.

#### First-Principles Solution
Alerting Query Concurrency

Alerting queries often involve filtering and aggregating large datasets based on specific conditions, such as threshold breaches or anomaly detection. Running these on a lake requires checking the following:

Query Execution Time: Large datasets can lead to lengthy query execution times.
Resource Contention: Multiple concurrent queries might compete for limited resources like CPU and memory.

To mitigate this:

Sharding: Divide the data into smaller, manageable chunks.
Query Optimization: Use indexes and partitioning to speed up query execution.
Resource Management: Ensure Trino is configured with sufficient resources or that it can dynamically scale out based on demand.
Freshness

Freshness refers to how quickly insights are derived from new data. This involves:

Data Ingestion Latency: How fast new data can be ingested and made available for querying.
Query Processing Time: How long it takes for queries to process the fresh data.

To ensure freshness:

Streaming Data Ingestion: Use a streaming ingestion mechanism like Apache Kafka or AWS Kinesis to ingest real-time data into the lake.
Incremental Queries: Design Trino queries to handle incremental updates efficiently, minimizing reprocessing of historical data.
Sub-second Dashboards

Sub-second dashboards require near-real-time processing and visualization. This involves:

Real-Time Data Processing: Support for real-time analytics without significant delay.
Visualization Performance: Ensuring the dashboard can render data quickly enough to meet user expectations.

To achieve sub-second dashboards:

Event-driven Architecture: Use event-driven architectures where relevant events trigger immediate processing and updates in the dashboard.
Materialized Views: Precompute and store frequently accessed views for faster querying.
Stress Test Scenarios
Concurrent Query Load Simulation: Simulate high-concurrency scenarios to measure performance under stress.
Data Freshness Validation: Verify that new data is being processed and available in the lake quickly enough.
Real-Time Visualization Testing: Run real-time dashboards and measure their responsiveness.

#### Closing Summary

Stress-testing shows that while a unified data architecture using Trino can handle most alerting queries, there are specific challenges with concurrency, freshness, and sub-second dashboards. To address these:

Implement sharding and query optimization.
Ensure efficient data ingestion and processing.
Use event-driven architectures for real-time visualization.

#### Common Mistakes/Follow-Ups
Overlooking Query Optimization: Not optimizing queries to handle high concurrency can lead to performance bottlenecks.
Neglecting Data Ingestion Latency: Poorly configured streaming ingestion can result in delayed insights.
Ignoring Real-Time Requirements: Failing to support real-time dashboards for critical use cases can degrade user experience.

**Comparison:**

| Resource | Optimal Configuration |
|----------|----------------------|
| CPU      | 8 vCPUs               |
| Memory   | 64 GB RAM            |
| Disk     | 500 GB SSD           |

**Code:**

```sql
-- Example query optimization using indexes and partitioning
CREATE INDEX idx_time ON lake_table(time_column);
CREATE TABLE IF NOT EXISTS alert_data WITH (storage_format = 'Parquet', location = 's3://alert-data/') AS
SELECT * FROM raw_data WHERE time_column > CURRENT_TIMESTAMP - INTERVAL 1 HOUR;
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `SELECT * FROM alert_data WHERE time_column > CURRENT_TIMESTAMP - INTERVAL 1 HOUR` | Alerted data within the last hour |

**Data:**

```json
{
  "data_source": "s3://raw-data/",
  "ingestion_method": "Kafka",
  "lake_storage_format": "Parquet"
}
```

**Flow:**

1. Yes
2. No
3. Data Ingestion
4. Real-time Data Processing
5. Is Data Fresh?
6. Lake Query Execution
7. Streaming Ingestion
8. Dashboard Update

