---
type: "practice"
difficulty: "Deep"
---

# How do you architect for the case where the AIOps model needs both streaming features (live) and slowly-changing enrichment (topology, ownership, deploy metadata) joined consistently? Where does the slowly-changing side live and how do you avoid a stale-join incident?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Log-first vs Processor-first Platforms](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Log-first_vs_Processor-first_Platforms.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this architecture design, we need to integrate real-time streaming features with slowly-changing enrichment data (e.g., topology, ownership, deployment metadata) in an AIOps system. The key challenge is ensuring consistency and avoiding stale joins while efficiently handling both types of data.

#### Triaging the Question

The question is essentially asking how to design a robust architecture that can handle real-time streaming data and slowly-changing data consistently without leading to stale join incidents. This involves understanding where the slowly-changing data should reside, how it should be updated, and what mechanisms ensure timely and accurate joins between these two types of data.

#### Assumptions
Real-time streaming features are time-sensitive and require immediate processing.
Slowly-changing enrichment data may have a longer update cycle but is critical for context and accuracy.
The system needs to handle millions of events per second with minimal latency.

#### First-Principles Solution

To achieve this, we need to design the architecture in such a way that real-time streaming features are processed as soon as possible while ensuring that slowly-changing data is always up-to-date for consistent joins.

Real-Time Streaming Data Handling:

Use Apache Kafka or similar platforms for ingesting and processing real-time data.
Implement low-latency stream processors using frameworks like Apache Flink or Spark Streaming to handle the streaming features in near-real time.

Slowly-Changing Enrichment Data Storage:

Store slowly-changing enrichment data in a database that supports high write throughput (e.g., Cassandra, RDS).
Ensure this data is periodically refreshed and updated but remains available for join operations.

#### Cons
istent Join Mechanism:

Use a hybrid approach where the real-time streaming features are joined with an up-to-date version of the slowly-changing data.
Implement a watermarking system to handle late-arriving events from the real-time stream, ensuring they do not disrupt the consistency of the join.

Data Synchronization:

Set up a periodic synchronization mechanism to keep the enrichment data current but with minimal impact on the streaming processing pipeline.
Use event time and watermarks in Kafka to handle out-of-order events gracefully.

Monitoring and Alerts:

Implement comprehensive monitoring and alerting systems to detect and resolve stale join incidents quickly.
Set up automated failover mechanisms to ensure high availability and resilience of both the streaming and enrichment data layers.

#### Closing Summary

To architect a system that integrates real-time streaming features with slowly-changing enrichment data, we need to carefully design the data flow, storage, and synchronization mechanisms. By leveraging modern stream processing frameworks and ensuring periodic updates while maintaining consistent joins, we can build a robust AIOps architecture that handles both types of data effectively.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to implement watermarking for late events in the streaming pipeline, leading to inconsistencies.
Mistake 2: Not setting up proper monitoring and alerting systems, causing delayed detection and resolution of issues.

#### Follow-Up
Question: How would you scale this architecture if the system needs to handle petabytes of data per day?

**Comparison:**

```
has
EVENT
STREAMING_DATA
```

**Code:**

```python
# Example code snippet for watermarking in Apache Flink
from pyflink.dataset import ExecutionEnvironment
from pyflink.table import StreamTableEnvironment, DataTypes
from pyflink.table.descriptors import Schema, OldCsv, FileSystem

env = ExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

t_env.connect(FileSystem().path('path/to/input')) \
    .with_format(OldCsv()
                 .field('event', 'STRING')
                 .field('timestamp', 'TIMESTAMP_LTZ(3)')) \
    .with_schema(Schema()
                 .field('event', DataTypes.STRING())
                 .field('timestamp', DataTypes.TIMESTAMP())) \
    .register_table_source('events')

t_env.connect(FileSystem().path('path/to/input')) \
    .with_format(OldCsv()
                 .field('data', 'STRING')
                 .field('timestamp', 'TIMESTAMP_LTZ(3)')) \
    .with_schema(Schema()
                 .field('data', DataTypes.STRING())
                 .field('timestamp', DataTypes.TIMESTAMP())) \
    .register_table_source('enrichment')

t_env.from_path('events') \
    .join(t_env.from_path('enrichment').assign_timestamp_and_watermark(lambda x: x.timestamp, 30)) \
    .where("events.timestamp > enrichment.timestamp - INTERVAL '10' MINUTE") \
    .execute().print()
```

**Example input / output:**

```
Event	Time Stamp	Data	Enrichment Time Stamp
log1	2023-10-01 12:00:01	data1	2023-10-01 11:59:01
log2	2023-10-01 12:05:01	data2	2023-10-01 11:59:51
Dry-Run
PLAINTEXT
Copy
Input Events:
log1, 2023-10-01T12:00:01Z, data1
log2, 2023-10-01T12:05:01Z, data2

Input Enrichment Data:
data1, 2023-10-01T11:59:01Z
data2, 2023-10-01T11:59:51Z

Watermark for Enrichment Data: 2023-10-01T12:00:00Z

Join Output:
log1, data1 (match with watermark)
log2, data2 (no match within watermark range, delayed processing needed)
```

**Data:**

```json
{
  "event": ["log1", "log2"],
  "timestamp": [Timestamp("2023-10-01 12:00:01"), Timestamp("2023-10-01 12:05:01")],
  "data": ["data1", "data2"],
  "enrichment_timestamp": [Timestamp("2023-10-01 11:59:01"), Timestamp("2023-10-01 11:59:51")]
}
```

**Flow:**

1. Event Source
2. Kafka Topic
3. Flink Stream Processor
4. Watermarking and Join Logic
5. Database for Enrichment Data
6. Periodic Sync Mechanism
7. Real-Time Processing Pipeline

