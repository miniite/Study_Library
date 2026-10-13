---
type: "practice"
difficulty: "Deep"
---

# Why might you choose ClickHouse or Apache Druid as a serving layer on top of the lake rather than querying Parquet directly with Trino for AIOps dashboards?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Telemetry Lake File Layout](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Telemetry_Lake_File_Layout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing observability data lakes for AIOps, choosing a serving layer like ClickHouse or Apache Druid over querying Parquet directly with Trino can significantly improve query performance and reduce latency, which are critical for real-time dashboards. The key difference lies in the architecture and design of these solutions.

Firstly, TL;DR: ClickHouse and Apache Druid are optimized for specific use cases—Druid excels at time-series data storage and real-time querying, while ClickHouse is a high-performance SQL database designed for analytical workloads. Both can offer faster query times compared to direct Parquet queries with Trino, especially when dealing with large datasets.

#### Triaging the Question

The question probes understanding of the trade-offs between different technologies in an AIOps context. Specifically, it asks why one might opt for ClickHouse or Apache Druid over querying a raw data lake (Parquet) directly using Trino. Key assumptions include:

The system needs to support real-time dashboards.
Performance and latency are critical.
Data is stored in Parquet format.

#### First Principles Solution

To understand this, we need to consider the characteristics of each technology:

ClickHouse:

Designed for high-performance analytics queries.
Optimized for SQL-based operations and complex aggregations.
Supports a wide range of data types and complex query patterns.

Apache Druid:

Specializes in time-series data storage and real-time querying.
Optimized for fast ingestion and low-latency queries.
Uses columnar storage for efficient data access.

Parquet + Trino:

Parquet is a columnar storage format designed for fast analytical query performance.
Trino (formerly Presto) is a distributed SQL query engine optimized for reading large datasets stored in various formats, including Parquet.
Step-by-Step Solution

Query Performance and Latency:

ClickHouse: Its in-memory columnar storage and efficient compression algorithms can provide sub-second response times for complex queries.
Apache Druid: Known for its real-time query capabilities with low-latency ingestion, making it ideal for dashboards that require near-instant updates.

Complexity of Queries:

For AIOps, queries often involve complex aggregations and joins over large datasets. ClickHouse handles such operations well.
Druid's time-series specific optimizations make it a better fit for continuous data streams and real-time analytics.

Data Ingestion Rate:

ClickHouse has robust ingestion capabilities but is not as specialized in high-throughput, low-latency scenarios as Druid.
Druid excels in handling high-frequency data ingestion from sources like logs or IoT devices.

Scalability and Resource Utilization:

Both ClickHouse and Druid are designed to scale horizontally.
However, the way they handle large datasets can differ—ClickHouse is optimized for analytical workloads, while Druid focuses on time-series data management.

#### Closing Summary

In summary, choosing between ClickHouse or Apache Druid over Parquet + Trino depends heavily on the specific requirements of your AIOps dashboard. Druid’s real-time capabilities and optimization for time-series data make it a strong choice when immediate visibility is crucial. ClickHouse, with its superior analytical query performance, is ideal for complex aggregations and joins over large datasets.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the specific requirements of real-time analytics vs. batch querying.

#### Follow-Up
Discussing how other technologies like TimescaleDB or InfluxDB could also be considered depending on the exact use case.

**Comparison:**

| Technology | Key Features                                  |
|------------|-----------------------------------------------|
| ClickHouse | High-performance SQL analytics queries       |
| Apache Druid | Real-time time-series data storage & querying |

**Code:**

```

-- Example ClickHouse query for real-time analytics SELECT count(), sum(value) FROM metrics WHERE time >= toStartOfDay(now()) - interval 1 hour;

CODE
Copy

```sql
-- Example Druid query for time-series data
SELECT mean(value), timestamp
FROM logs
GROUP BY time(timestamp) EVERY 5 minutes
WHERE date >= now() - duration(1, HOURS);
```

