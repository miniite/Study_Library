---
type: "practice"
difficulty: "Expert"
---

# Your telemetry estate ingests 20PB/day of metrics logs traces and events. Architect the org-wide observability data platform: what lives in a hot columnar store versus a cheap object-store lake versus a warehouse for post-hoc analytics, and how do you draw the tiering boundary by query pattern?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Centralized vs Mesh Observability Data](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Centralized_vs_Mesh_Observability_Data.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an observability data platform capable of handling 20PB/day of telemetry data, we need to strategically allocate different types of storage based on their characteristics and usage patterns. The goal is to ensure efficient querying, post-hoc analytics, and cost-effectiveness while maintaining high availability and reliability.

The key components in this architecture are a hot columnar store, an object-store lake, and a warehouse for post-hoc analytics. We'll tier the data based on query patterns and the nature of the queries being made.

#### Triage
Understanding the Question
Objective: Design an observability data platform that can handle 20PB/day of telemetry data.
Key Components: Hot columnar store, object-store lake, warehouse for analytics.
Query Patterns: Distinguishing between real-time querying and post-hoc batch queries.

#### Assumptions
The organization requires both low-latency access to recent data (e.g., for alerting) and the ability to perform historical analysis.
The platform must support various types of telemetry data: metrics, logs, traces, and events.
Cost optimization is a primary concern due to the high volume of data.

#### First-Principles Solution

Hot Columnar Store:

Purpose: Real-time querying and alerting on recent data.

#### Example
Technologies: Apache Druid, InfluxDB, TimeSeries databases.
Data Retention Policy: Data retained for up to 24 hours (or less if performance is critical).

Object-Store Lake:

Purpose: Long-term storage of raw and semi-processed telemetry data.

#### Example
Technologies: Amazon S3, Azure Blob Storage, Google Cloud Storage.
Data Retention Policy: Data retained for 1 year to 5 years (depending on regulatory requirements).
Compression: Utilize compression techniques like Snappy or Zstandard to reduce storage costs.

Warehouse for Analytics:

Purpose: Batch processing and post-hoc analytics.

#### Example
Technologies: Presto, Snowflake, Apache Hive, Delta Lake.
Data Retention Policy: Data retained indefinitely but optimized for querying and analysis.
Query Patterns: Support complex SQL queries with JOINs, GROUP BYs, and aggregations.
Tiering Boundary by Query Pattern:

Real-time Queries (e.g., for alerting):

Access the hot columnar store.
These queries require low latency and high availability.

Batch Queries (e.g., for monthly reports, trend analysis):

Use the warehouse for analytics.
These queries can tolerate higher latencies but require complex processing capabilities.

Archival Data:

Store raw data in the object-store lake for long-term archival and compliance purposes.
Periodically refresh the warehouse with recent data to support more complex queries.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes

Failing to distinguish between real-time and batch queries, leading to inefficient use of resources.
Over-provisioning storage in the hot columnar store, increasing costs without a corresponding benefit.

#### Follow-Ups

Discuss data partitioning strategies for both hot and cold tiers.
Explore additional cost optimization techniques like leveraging serverless analytics platforms or optimizing query execution plans.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Hot Columnar Store] --> B1{Query Type};
    B1 --> C1[Real-time Alerts];
    B1 --> C2[Batch Reports];

    D[Object-Store Lake] --> E1{Query Type};
    E1 --> F1[Long-term Archival];

    G[Warehouse for Analytics] --> H1{Query Type};
    H1 --> I1[Post-hoc Analysis];
```

**Example input / output:**

| Query Type | Hot Columnar Store | Object-Store Lake | Warehouse for Analytics |
|------------|--------------------|------------------|-------------------------|
| Real-time Alerts | Druid, InfluxDB | N/A | N/A |
| Batch Reports | N/A | N/A | Presto, Snowflake |

**Data:**

```yaml
# Example Data Retention Policies
retention_policy:
  hot_columnar_store: 24 hours
  object_store_lake: 5 years
  warehouse_for_analytics: indefinitely

# Example Compression Settings
compression:
  object_store_lake: snappy
```

