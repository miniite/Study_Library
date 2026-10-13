---
type: "practice"
difficulty: "Expert"
---

# You must store time-series metrics at 1M points/min and query the last hour per host. Compare a Cassandra TWCS bucketed model vs a DynamoDB composite-key model. Which do you pick and why?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Wide Column Design and Migration](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Wide_Column_Design_and_Migration.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compare a Cassandra Time Window Column (TWCS) bucketed model with a DynamoDB composite-key model for storing time-series metrics at 1M points per minute and querying the last hour per host, we need to consider several factors including data distribution, query performance, write throughput, cost, and operational complexity. Cassandra's TWCS is designed for high write throughput and time-series data, while DynamoDB offers managed NoSQL capabilities with flexible schemas but may have limitations in handling large-scale time-series workloads.

Firstly, let’s break down the requirements:

Data Volume: 1M points per minute.
Query Requirement: Retrieve the last hour of metrics per host.

#### Triaging the Question

The question primarily probes your understanding of wide-column NoSQL database models and their trade-offs in handling time-series data. We need to consider both Cassandra’s TWCS bucketing strategy and DynamoDB’s composite-key approach, evaluating them based on write scalability, query efficiency, cost, and operational overhead.

#### Assumptions
Metrics: Each metric has a timestamp.
Hosts: There are many hosts generating metrics.
Query Frequency: We need to frequently fetch the last hour of data for each host.

#### First-Principles Solution
Cassandra Time Window Column (TWCS) Model

Cassandra is designed with a distributed architecture and offers a TWCS model specifically for time-series data. With this model, we can efficiently store and query time-series data by bucketing it based on time intervals.

Bucketing: Divide the timestamp into buckets of a predefined size (e.g., 5 minutes or 1 hour). Each bucket will contain data points within that time interval.
Row Key: Use a composite row key consisting of the host and timestamp bucket.
Column Name: The column name can be the specific metric, and the value is the actual measurement.

For example:

CASSANDRA
CREATE TABLE metrics (
    host text,
    bucket_time timestamp,
    metric_name text,
    value double,
    PRIMARY KEY (host, bucket_time)
);

Write Performance: Cassandra handles writes efficiently due to its distributed nature and write-ahead logging. The write performance is optimal when data is written in a sequential manner.

Query Performance: Queries like "get the last hour of metrics for host X" can be optimized by using secondary indexes on bucket_time within the same partition (host).

DynamoDB Composite-Key Model

DynamoDB offers managed, flexible schema NoSQL capabilities. For time-series data, we can use a composite primary key with a timestamp as one part and possibly a unique identifier for the host.

Table Structure: Use a composite primary key of timestamp and host.
Query Optimization: Utilize DynamoDB's secondary indexes to facilitate efficient query operations.

For example:

JSON
{
  "Host": "host-001",
  "Timestamp": 1643724000, // Unix timestamp in seconds
  "MetricName": "cpu_usage",
  "Value": 85.9
}

Write Performance: DynamoDB also handles writes efficiently but might have limitations if the write throughput exceeds its capacity.

Query Performance: Queries like "get the last hour of metrics for host X" can be optimized using secondary indexes on timestamp and filtering by the last hour.

Comparative Analysis

Cassandra (TWCS):

#### Pros
High write scalability.
Efficient time-series data storage and querying.
Low operational overhead due to its distributed nature.

#### Cons
Requires careful bucketing strategy design for optimal performance.
Potentially higher cost in terms of storage when not using compression effectively.

DynamoDB (Composite-Key):

#### Pros
Simplified setup and management with a managed service.
Flexible schema, easy to scale without downtime.

#### Cons
Limited by DynamoDB’s write throughput capacity.
Potential complexity in designing secondary indexes for optimal performance.

#### Closing Summary

Given the high volume of writes (1M points per minute) and frequent read queries (last hour), Cassandra with a TWCS model is likely the better choice. Its distributed architecture, efficient handling of time-series data, and ability to scale out well make it more suitable for this workload compared to DynamoDB.

#### Common Mistakes/Follow-Ups
Not considering write scalability: Both systems handle writes efficiently, but you need to ensure your write throughput is managed properly.
Ignoring query optimization: Ensure that secondary indexes or partitioning strategies are correctly implemented in both systems.
Cost considerations: Evaluate long-term storage costs and operational overhead for each system.

**Comparison:**

```
Timestamp	Host	MetricName	Value
1643725200	host-001	cpu_usage	85.9
1643725201	host-001	mem_usage	90.1
1643725202	host-001	disk_io	42
```

**Code:**

```
CASSANDRA
Copy
CREATE TABLE metrics (
    host text,
    bucket_time timestamp,
    metric_name text,
    value double,
    PRIMARY KEY (host, bucket_time)
);
```

**Example input / output:**

```text
Input:
- Insert metrics for host 'host-001' with various timestamps and metric names.

Output:
- Retrieve the last hour of data for a specific host.
```

**Data:**

```json
{
  "Host": "host-001",
  "Timestamp": 1643724000, // Unix timestamp in seconds
  "MetricName": "cpu_usage",
  "Value": 85.9
}
```

**Flow:**

1. Yes
2. No
3. Insert Metrics
4. Is Bucket Time Interval Met?
5. Write to Cassandra TWCSTable
6. Adjust Bucketing Strategy andRetry
7. Query Last Hour Data for Host
8. Return Results or Error

