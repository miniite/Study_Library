---
type: "practice"
difficulty: "Deep"
---

# Your averaged pipeline-latency dashboard looks fine but one downstream team reports stale data for their partition. Explain why aggregate metrics hide per-partition tail behavior and how you'd instrument per-partition or per-tenant freshness.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Aggregation Hiding Tail Problems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Aggregation_Hiding_Tail_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this issue, we need to understand that aggregate metrics can give a misleading overall view of system performance, particularly when there are significant differences in tail behavior across different partitions or tenants. In distributed systems, per-partition or per-tenant freshness instrumentation is crucial for identifying and resolving issues like stale data.

#### Triaging the Question

The question asks us to explain why aggregated pipeline latency metrics might appear fine while a downstream team reports staleness for their specific partition or tenant. We need to delve into the details of how distributed systems operate, the trade-offs between aggregation and granularity, and how we can instrument our system to ensure freshness across all partitions.

Step-by-Step Solution

Understand Aggregate Metrics: Aggregated metrics provide an overview of the overall system performance by summarizing data from multiple components or tenants. For instance, if you are tracking pipeline latency, your dashboard might show an average latency over all partitions or tenants. However, this aggregate view can mask significant variability and outliers.

Tail Behavior vs. Average: Tail behavior refers to the longer end of a distribution, often characterized by extreme values that occur infrequently but significantly impact overall performance and user experience. Aggregating these tail latencies with other data points can result in an average latency that doesn't reflect the worst-case scenarios for specific partitions or tenants.

Instrumentation Requirements: To address the issue of staleness, we need to instrument our system at a finer granularity—per-partition or per-tenant. This involves tracking individual metrics for each component and ensuring that each partition or tenant has its own set of freshness checks.

Implementing Per-Partition Freshness Checks:

Data Ingestion Monitoring: Track the time between data ingestion and processing completion for each partition.
Processing Latency: Measure the latency at various stages of processing within a partition to identify bottlenecks.
Cache Validity: Ensure that cached data is valid and up-to-date, especially in systems where stale data can propagate.
Health Checks: Implement regular health checks for each partition or tenant to detect issues early.

Monitoring Freshness with Watermarks: Use watermarks to track the freshness of data streams within a partition. Watermarks help ensure that data is only processed once and can be used to validate the staleness of data in real-time.

Alerting and Notifications: Set up alerts for deviations from expected freshness levels. This could involve setting thresholds for latency or frequency of updates, and triggering notifications when these thresholds are exceeded.

#### Closing Summary

In summary, aggregate metrics can hide critical issues within distributed systems by averaging out extreme tail latencies. To ensure data freshness across all partitions or tenants, it's essential to implement detailed instrumentation that tracks individual components and their performance. By doing so, we can identify and resolve staleness issues more effectively, ensuring reliable and timely data delivery.

#### Common Mistakes
and Follow-Ups
Overreliance on Aggregates: Failing to complement aggregate metrics with granular per-partition or per-tenant checks.
Insufficient Granularity in Freshness Checks: Not implementing detailed monitoring for each component within a partition, leading to undetected staleness issues.
Ignoring Real-Time Processing Delays: Overlooking the impact of real-time processing delays on data freshness, especially in time-sensitive applications.

If you have any specific follow-ups or need further details on certain aspects, feel free to ask!

**Comparison:**

| Partition ID | Name    | Ingested Time                | Processed Time               |
|--------------|---------|-----------------------------|------------------------------|
| 1            | partition-a | 2023-09-25T14:30:00Z        | 2023-09-25T14:35:00Z        |
| 2            | partition-b | 2023-09-26T08:45:00Z        | 2023-09-26T17:00:00Z        |
| 3            | partition-c | 2023-09-27T09:15:00Z        | 2023-09-27T09:25:00Z        |

**Code:**

```python
# Example Python code for per-partition latency tracking
import pandas as pd

def track_partition_latency(partition_data):
    # Track latency for each partition
    latencies = [data['processed_time'] - data['ingested_time'] for _, data in partition_data.iterrows()]
    return pd.DataFrame({'partition': partition_data.index, 'latency': latencies})
```

**Example input / output:**

```text
Input:
- Partition Data with Ingestion and Processing Times

Output:
- DataFrame with Partition ID and Latency for Each Partition

Example Input:
{
  "partitions": [
    {"id": 1, "name": "partition-a", "data_ingested_time": "2023-09-25T14:30:00Z", "data_processed_time": "2023-09-25T14:35:00Z"},
    {"id": 2, "name": "partition-b", "data_ingested_time": "2023-09-26T08:45:00Z", "data_processed_time": "2023-09-26T17:00:00Z"},
    {"id": 3, "name": "partition-c", "data_ingested_time": "2023-09-27T09:15:00Z", "data_processed_time": "2023-09-27T09:25:00Z"}
  ]
}

Example Output:
+-----------+----------+
| partition | latency  |
+===========+==========+
|         1 | 00:05:00 |
|         2 | 08:15:00 |
|         3 | 00:10:00 |
+-----------+----------+
```

**Data:**

```json
{
  "partitions": [
    {"id": 1, "name": "partition-a", "data_ingested_time": "2023-09-25T14:30:00Z", "data_processed_time": "2023-09-25T14:35:00Z"},
    {"id": 2, "name": "partition-b", "data_ingested_time": "2023-09-26T08:45:00Z", "data_processed_time": "2023-09-26T17:00:00Z"},
    {"id": 3, "name": "partition-c", "data_ingested_time": "2023-09-27T09:15:00Z", "data_processed_time": "2023-09-27T09:25:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Data Ingested
4. Is Data Processed?
5. Record Latency
6. Queue for Processing
7. Alert if Latency ExceedsThreshold
8. Retry or Queue for Later

