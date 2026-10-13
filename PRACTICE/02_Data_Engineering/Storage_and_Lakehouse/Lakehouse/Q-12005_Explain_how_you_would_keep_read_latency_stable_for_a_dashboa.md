---
type: "practice"
difficulty: "Deep"
---

# Explain how you would keep read latency stable for a dashboard while a large backfill writes to the same Iceberg table.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lakehouse Fundamentals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that read latency remains stable for a dashboard during a large backfill operation on an Iceberg table, we need to carefully manage data write operations and optimize read paths. The key challenge is to minimize the impact of ongoing writes on existing queries, ensuring that dashboards can continue to fetch timely data without significant delays.

#### Triaging the Question

The question revolves around maintaining real-time dashboard performance during a large-scale backfill operation on an Iceberg table. This involves understanding how Iceberg handles concurrent read and write operations and exploring strategies to mitigate the impact of writes on reads, such as partitioning, caching, and data versioning.

#### Assumptions
The dashboard queries are read-intensive.
The backfill operation is a one-time, large-scale update that significantly impacts the table's size and schema.
The Iceberg table supports vectorized operations and lazy compaction for efficient write processing.

#### First-Principles Solution

To maintain stable read latency, we can implement several strategies:

Partitioning and Staging:

Use partitioning to isolate the backfill operation from active queries. For example, split the data into smaller partitions that can be processed in parallel.
Introduce a staging area where new data is written before being merged with existing partitions.

Caching:

Implement caching mechanisms to store recent query results. This reduces the need for frequent reads and helps in maintaining consistent read performance.

Versioning and Compaction:

Use versioned tables to keep track of historical data versions, allowing dashboards to fetch from a specific version that is not undergoing writes.
Schedule compaction operations during off-peak times or use lazy compaction settings to minimize the impact on read performance.

Query Routing and Sharding:

Implement query routing to route reads to specific partitions or nodes where the latest data is not being written.
Use sharding techniques to distribute queries across multiple instances, ensuring that no single node bears too much load.

Monitoring and Alerting:

Set up monitoring to detect performance degradation early and alert when read latency exceeds acceptable thresholds.
Use A/B testing to validate the effectiveness of different strategies before full-scale deployment.

#### Closing Summary

By employing partitioning, caching, versioning, query routing, and robust monitoring, we can ensure that dashboards continue to operate with minimal disruption during large backfill operations. Each strategy addresses specific aspects of read/write concurrency, ensuring that performance remains stable even under high write load conditions.

#### Common Mistakes/Follow-Ups
Ignoring Write-Read Concurrency: Failing to partition or isolate writes from reads can lead to significant delays in dashboard queries.
Overcomplicating Compaction: Not understanding how lazy compaction works and when to use it can lead to unnecessary complexity and potential performance issues.
Lack of Monitoring: Without proper monitoring, identifying and addressing performance bottlenecks becomes challenging.

For follow-ups, we might explore more advanced techniques like distributed caching or hybrid storage solutions that combine cold and hot data storage for even better performance.

**Code:**

```python
# Example of partitioning a large backfill operation
def backfill_partitioned(table):
    partitions = get_partitions_from_table(table)
    for part in partitions:
        backfill(partition=part, table=table)

def backfill(partition, table):
    # Logic to read and process data into a staging area
    stage_data(partition, table)
    # Merge staged data with the main table
    merge_staged_data(partition, table)
```

**Example input / output:**

| Partition | Start Time (UTC) | End Time (UTC) |
|-----------|-----------------|---------------|
| partition1 | 2023-10-01T08:00 | 2023-10-01T14:00 |
| partition2 | 2023-10-01T15:00 | 2023-10-01T21:00 |

**Data:**

```json
{
  "partitions": [
    {"name": "partition1", "data_size": 50MB},
    {"name": "partition2", "data_size": 75MB}
  ]
}
```

