---
type: "practice"
difficulty: "Expert"
---

# How would you migrate a Lambda architecture with separate batch and speed layers to a single Kappa streaming pipeline and what do you give up?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Choosing Processing Engines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Batch_vs_Streaming_Features/Choosing_Processing_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To migrate from a Lambda architecture using separate batch and speed layers to a Kappa streaming pipeline, we need to understand that the Lambda architecture processes data in two stages—batch processing for historical data and real-time processing for current data. The goal of migrating this setup to a Kappa architecture is to consolidate both batch and stream processing into one scalable system that can handle both types of workloads efficiently.

#### Triage
Understanding the Question

The interviewer is probing your understanding of distributed systems, stream processing frameworks like Kafka and Flink, and the trade-offs between different architectural styles. The key assumptions here are:

The current Lambda architecture processes data in two distinct layers.
There's a need to migrate this setup while maintaining operational simplicity and scalability.

#### First Principles Solution
Understand the Requirements: Identify what operations are currently handled by the batch and speed layers of the Lambda architecture.
Choose a Streaming Framework: Select a suitable streaming framework like Apache Kafka or Apache Flink, which can handle both historical data processing and real-time data processing.
Design the Pipeline: Implement a single pipeline that uses the chosen framework to process both types of data streams.
Ensure Fault Tolerance and Scalability: Ensure that the new system can handle failures gracefully and scale according to the load.
Detailed Solution
Step 1: Understand the Requirements
Batch Layer: Processes historical data periodically (e.g., hourly, daily) for generating aggregates or insights.
Speed Layer: Processes real-time data streams to provide near-real-time updates (e.g., micro-batch processing).
Step 2: Choose a Streaming Framework

For this migration, we'll use Apache Kafka as the streaming platform due to its robustness and wide adoption. We will integrate it with Apache Flink for processing both historical and real-time data.

Historical Data Source
Kafka Topic
Real-Time Data Source
Kafka Topic
Flink Batch Job
Flink Stream Job
Data Store for Aggregates
Real-time Update Store
Step 3: Design the Pipeline

Historical Data Processing:

Use Flink's batch processing capabilities to read from Kafka, process historical data, and store aggregated results in a data store.

Real-Time Data Processing:

Use Flink’s stream processing capabilities to handle real-time data streams, perform necessary transformations, and update the real-time store.
Step 4: Ensure Fault Tolerance and Scalability
Fault Tolerance: Configure Kafka with multiple replicas for high availability. Use Flink's checkpointing mechanism to ensure state recovery.
Scalability: Design the pipeline to scale out based on demand by adding more Kafka brokers or Flink task managers.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Migrating from a Lambda architecture to a Kappa streaming pipeline involves consolidating batch and stream processing into one scalable system. The key is choosing the right tools, understanding data flow, and ensuring fault tolerance and scalability. Common mistakes include:

Overlooking real-time state management.
Not considering the trade-offs between micro-batch and true real-time processing.

#### Follow-ups
might involve discussing specific implementation details or comparing costs and benefits of different technologies.

**Comparison:**

| Kafka Topic | Partitions | Replication Factor |
|-------------|------------|--------------------|
| historical_data | 3          | 2                  |
| real_time_data | 5          | 1                  |

**Code:**

```python
# Example Flink Batch Job
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import TableConfig, DataTypes, StreamTableEnvironment

env = ExecutionEnvironment.get_execution_environment()
t_config = TableConfig()
table_env = StreamTableEnvironment.create(env, t_config)

# Reading from Kafka topic
kafka_source_ddl = """
CREATE TABLE kafka_batch (
    event_id INT,
    user_id BIGINT,
    item_id BIGINT,
    event_time TIMESTAMP(3),
    WATERMARK FOR event_time AS event_time - INTERVAL '5' SECOND
) WITH ('connector.type'='kafka', ...)

# Processing logic for batch job
batch_job_ddl = """
INSERT INTO aggregated_data
SELECT user_id, COUNT(*)
FROM kafka_batch
GROUP BY TUMBLE(event_time, INTERVAL '10' MINUTE);
"""

table_env.execute_sql(batch_job_ddl)
```

**Example input / output:**

```text
Input (Historical Data):
Event ID: 1, User ID: 100, Item ID: 200, Event Time: 2023-10-01 14:00:05

Output (Aggregated Data):
User ID: 100, Count: 5
```

**Data:**

```json
{
  "kafka_topic": {
    "name": "historical_data",
    "partitions": 3,
    "replication_factor": 2
  },
  "kafka_stream_topic": {
    "name": "real_time_data",
    "partitions": 5,
    "replication_factor": 1
  }
}
```

**Flow:**

1. Historical Data Source
2. Kafka Topic
3. Real-Time Data Source
4. Kafka Topic
5. Flink Batch Job
6. Flink Stream Job
7. Data Store for Aggregates
8. Real-time Update Store

