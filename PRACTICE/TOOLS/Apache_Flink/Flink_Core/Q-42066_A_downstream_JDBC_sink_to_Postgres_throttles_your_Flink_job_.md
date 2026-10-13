---
type: "practice"
difficulty: "Deep"
---

# A downstream JDBC sink to Postgres throttles your Flink job and lag climbs. How do you localize and fix the backpressure?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Checkpoint Overhead](<../../../../TOOLS/Apache_Flink/Flink_Core/Checkpoint_Overhead.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a downstream JDBC sink to PostgreSQL throttles an Apache Flink stream processing job, causing lag to climb, we need to diagnose the root cause of this backpressure and implement corrective measures. The key steps involve understanding the flow, identifying bottlenecks, tuning the configuration, and ensuring data consistency.

First, let’s triage the problem. We are dealing with a scenario where Flink's streaming job is being throttled by the JDBC sink to PostgreSQL, leading to increased lag. This suggests that the issue lies somewhere in the interaction between Flink and the database. Possible causes include:

Database Performance Issues: The database might be under high load or facing concurrency issues.
Flink Configuration Settings: The Flink job's configurations related to parallelism, batch size, and other backpressure settings may need tuning.
Network Latency: Network issues could contribute to the problem.

To diagnose this, we will follow a systematic approach from first principles:

Review Flink Job Logs:

Check if there are any warnings or errors related to the JDBC sink or database operations.
Look for indications of backpressure such as RateExceededException logs.

Analyze Database Performance Metrics:

Use PostgreSQL’s built-in tools (e.g., pg_stat_activity, pg_stat_all_tables) to check if there are any queries running long or blocking others.
Monitor the database's CPU and memory usage to ensure it is not overloaded.

Check Flink Job Parameters:

Ensure that the Flink job’s parallelism matches the number of available resources.
Verify the batch size in the JDBC sink configuration; a too small batch size can lead to frequent writes, increasing latency.
Check if the setAutoCommit property is set correctly and consider using transactional writes for better performance.

Tune Backpressure Mechanisms:

Increase the checkpoint interval if it’s currently too low.
Adjust the degree of parallelism in the Flink job to distribute the load more evenly.

#### Cons
ider configuring the maxParallelism setting to ensure that no single task is overwhelmed by data.

Implement Retry Mechanisms and Error Handling:

Add retry logic for failed database operations to avoid permanent failure points.
Ensure proper error handling in Flink to manage transient failures gracefully.

By systematically addressing these areas, we can identify the root cause of backpressure and implement appropriate fixes. Common mistakes include over-tuning without understanding the underlying issue or making changes that may introduce new issues.

**Comparison:**

| Flink Job Parameters | Value        |
|---------------------|--------------|
| Parallelism         | 4            |
| Batch Size          | 10           |
| Checkpoint Interval | 5 minutes     |

**Code:**

```java
// Example configuration for a JDBC sink in Flink
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
DataStream<Tuple2<String, String>> sourceData = ...; // Your data stream

sourceData.addSink(new FlinkJdbcBatchSink(
    "jdbc:postgresql://localhost:5432/mydb",
    "INSERT INTO my_table (id, value) VALUES (?, ?)",
    new RowTypeInfo(String.class, String.class),
    new BatchStatementExecutor<>(10), // Batch size of 10
    new Properties(), // Additional JDBC properties if needed
    5 * 60 * 1000)); // Checkpoint interval of 5 minutes
```

**Example input / output:**

```text
Input:
sourceData = [
  ("row1_id", "value1"),
  ("row2_id", "value2"),
  ...
]

Output:
sinkOutput = [
  {"id": "row1_id", "result": true},
  {"id": "row2_id", "result": false, "error": "Database is unresponsive"}
]
```

**Data:**

```json
{
  "jdbcUrl": "jdbc:postgresql://localhost:5432/mydb",
  "insertQuery": "INSERT INTO my_table (id, value) VALUES (?, ?)",
  "batchSize": 10,
  "checkpointIntervalMs": 300000,
  "additionalProperties": {
    "user": "postgres",
    "password": "mysecretpassword"
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Review Flink Job Logs
5. Check forRateExceededException
6. Adjust Batch Size andParallelism
7. Analyze Database PerformanceMetrics
8. Migrate to Transactional Writes
9. Tune BackpressureMechanisms
10. Implement Retry Logic
11. End

