---
type: "practice"
difficulty: "Deep"
---

# Explain how log-based CDC preserves ordering and at-least-once delivery from a source DB to a lakehouse and how you'd deduplicate and apply upserts/deletes in order so the derived table converges to the source.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Keeping Indexes in Sync via CDC](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Keeping_Indexes_in_Sync_via_CDC.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain how log-based CDC (Change Data Capture) preserves ordering and ensures at-least-once delivery from a source database to a lakehouse, and to address deduplication and upserts/deletes in order so that the derived table converges to the source, let’s break down the problem step by step.

Firstly, log-based CDC leverages transaction logs (like WAL - Write-Ahead Logs) of the source database to capture changes efficiently. The core challenge is ensuring that all updates are captured and processed in the correct order, even when they occur concurrently or across distributed systems. This involves handling both insertions and deletions, and applying upserts appropriately while maintaining a consistent view.

#### Triaging the Question

The question probes an understanding of:

Ordering and Delivery Guarantees: Ensuring that changes are delivered in the correct order.
At-Least-Once Delivery: Guaranteeing that no change is lost, but allowing for potential duplicates.
Deduplication and Upserts/Deletes: Handling updates without causing inconsistencies.

#### Assumptions
and Approach
We assume a robust distributed system with multiple nodes processing the logs concurrently.
The source database uses transactional consistency models like ACID (Atomicity, Consistency, Isolation, Durability).
The lakehouse environment supports materialized views or updatable tables for efficient processing.

#### First-Principles Solution

Log-Based CDC:

Log Format: Use the source database’s native transaction logs to capture changes.
Ordering: Ensure that all log entries are timestamped and ordered by the transaction start time. This helps in processing updates in the correct sequence.

Distributed Processing:

Eventual Consistency: Use a distributed event-driven architecture like Apache Kafka or AWS Kinesis to ingest logs.
Ordering Guarantees: Leverage stream processors (like Apache Flink, Apache Spark Streaming) that support exactly-once processing semantics.

Processing Pipeline:

Capture and Enqueue Logs: Ingest the logs into a message queue where each log entry includes metadata like source, target table, and timestamp.
Eventual Processing: Process these messages in order, ensuring no out-of-order events. This can be achieved by using ordered queues or partitioning strategies.

Deduplication:

Hash-Based Deduplication: Use a hash of the key fields to identify duplicates. Store hashes in a lookup table to quickly check for existing records.
Timestamp Comparison: For complex cases, compare timestamps with the last update time in the lakehouse.

Upserts and Deletes:

Upsert Logic: When an upsert is detected (insert or update), ensure that only one operation is applied per key. This can be managed using transactional semantics.
Delete Handling: Mark records for deletion based on the log entry, ensuring they are removed once confirmed in the lakehouse.

Convergence:

Materialized Views: Create materialized views or updatable tables in the lakehouse that reflect the latest state of the source database.
Incremental Refreshes: Regularly refresh these views to ensure convergence with minimal latency.

#### Closing Summary
and Common Mistakes
Mistake 1: Missing Ordering Guarantees: Failing to maintain strict ordering can lead to out-of-sequence updates, causing data inconsistencies.
Mistake 2: Incomplete Deduplication Logic: Not handling hash collisions or duplicate keys correctly can result in incorrect state representation.
Mistake 3: Ignoring Transaction Semantics: Overlooking the importance of transactional semantics can lead to race conditions and data corruption.

#### Follow-Ups
How would you handle partial failures in the distributed processing pipeline?
What strategies do you use to optimize performance while maintaining consistency?

**Comparison:**

| Key | Value                             | Timestamp          |
|-----|----------------------------------|--------------------|
| user_1 | {"name": "Alice", "age": 30}   | 2023-10-04T15:19:30Z |

**Code:**

```python
# Example Flink Job for CDC Processing
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

t_env.connect(FileSystem().path('/logs')) \
    .with_format(OldCsv()
                 .field('key', 'STRING')
                 .field('value', 'STRING')
                 .field('timestamp', 'TIMESTAMP')) \
    .with_schema(Schema()
                 .field('key', DataTypes.STRING())
                 .field('value', DataTypes.STRING())
                 .field('timestamp', DataTypes.TIMESTAMP())) \
    .register_source('logSource')

t_env.connect(FileSystem().path('/output')) \
    .with_format(OldCsv()
                 .field('key', 'STRING')
                 .field('value', 'STRING')
                 .field('timestamp', 'TIMESTAMP')) \
    .with_schema(Schema()
                 .field('key', DataTypes.STRING())
                 .field('value', DataTypes.STRING())
                 .field('timestamp', DataTypes.TIMESTAMP())) \
    .register_sink('logSink')

t_env.scan('logSource') \
    .filter("not exists(key, value)")  # Example filter
    .insert_into('logSink')
```

**Example input / output:**

```text
Input:
Key: user_1, Value: {"name": "Alice", "age": 30}, Timestamp: 2023-10-04T15:19:30Z

Output:
Key: user_1, Value: {"name": "Alice", "age": 30}, Timestamp: 2023-10-04T15:19:30Z
```

**Data:**

```json
{
  "key": "user_id",
  "value": "user_data",
  "timestamp": "2023-10-04T15:19:30Z"
}
```

**Flow:**

1. Yes
2. No
3. Log Ingestion
4. Ordering?
5. Process Logs
6. Reorder and Process Logs
7. Apply Upserts/Deltes
8. Deduplicate and ApplyUpserts/Deltes
9. Converge to Source State

