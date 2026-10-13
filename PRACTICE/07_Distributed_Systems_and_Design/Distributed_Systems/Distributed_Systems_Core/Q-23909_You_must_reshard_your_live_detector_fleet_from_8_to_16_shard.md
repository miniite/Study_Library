---
type: "practice"
difficulty: "Deep"
---

# You must reshard your live detector fleet from 8 to 16 shards to handle growth without losing windowed baseline state or missing detection. Walk me through the dual-assignment, state-handoff, backfill, and cutover plan and how you'd roll it back.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Backfills](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Backfills.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Resharding a live detector fleet from 8 to 16 shards is a critical task that requires careful planning to ensure no loss of windowed baseline state or detection capabilities. The process involves dual-assignment, state-handoff, backfill, and cutover steps, each with its own set of complexities. Let's break this down step by step.

First, we need to understand the current system's structure and how the data is distributed across shards. Assuming a typical distributed system architecture where each shard has a portion of the dataset and associated state, the dual-assignment phase involves simultaneously managing both old and new shard configurations to ensure no disruption in service.

#### Triage
What the Question Is Asking

The question probes your ability to manage a complex resharding operation while maintaining operational stability. Key points include:

Dual-Assignment: How to handle data and state during the transition.
State-Handoff: Efficiently transferring state from old shards to new ones.
Backfill: Ensuring that all historical data is processed correctly in the new setup.
Cutover Plan: A smooth switch from the old to the new configuration.

#### Assumptions
The current system uses consistent hashing for shard assignment.
Each shard stores both historical and real-time data, along with associated state (windowed baseline).
We have a monitoring system in place to track performance metrics during resharding.
Step-by-Step Solution
1. Dual-Assignment Phase

Objective: Simultaneously manage the old and new shards without service disruption.

Preparation: Ensure both the old and new shard configurations are ready. This includes creating the necessary shards, setting up the state transfer pipeline, and configuring the monitoring system.
Monitoring: Continuously monitor the health of the current system to detect any issues early.
2. State-Handoff Phase

Objective: Transfer state from the old shards to the new ones.

Data Transfer Strategy: Use a combination of batch processing (for historical data) and real-time processing (for ongoing data). For example, use Apache Kafka for streaming data and Apache Spark for bulk data migration.
State Consistency: Ensure that the state is consistent across both old and new shards. This might involve creating checkpoints during the transfer process to prevent data loss.
3. Backfill Phase

Objective: Process all historical data in the new setup to ensure no windowed baseline state is lost.

Data Retention: Implement a mechanism to retain enough historical data for accurate windowing. For instance, using time-series databases like InfluxDB.
Incremental Processing: Use incremental backfills to process only recent data that was missed during the transition.
4. Cutover Plan

Objective: Smoothly switch from the old shards to the new ones without disrupting service.

Rolling Update Strategy: Perform the cutover in a rolling manner, updating one node at a time to minimize downtime and ensure consistent state across all nodes.
Final Checks: Verify that all data and state have been successfully transferred and processed. Use automated tests and manual checks to confirm the transition.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The resharding process is complex but crucial for scaling the system without losing critical functionality. Key challenges include managing state consistency, ensuring no service disruption during cutover, and validating the correctness of the new setup.

#### Common Mistakes

Ignoring State Consistency: Failing to ensure that old and new states are fully consistent can lead to data loss or incorrect detections.
Insufficient Monitoring: Lack of real-time monitoring can result in undetected issues during the transition.
Poor Backfill Strategy: Inadequate handling of historical data can lead to incorrect windowed baselines.

#### Follow-Ups
What tools would you use for state transfer and backfill?
How would you handle potential bottlenecks in the data processing pipeline?
Can you discuss a specific scenario where this resharding approach might fail, and how you would mitigate it?

**Comparison:**

```
Source Shard Setup
State Transfer
Monitoring
Incremental Backfill
Cutover Plan
Successful Transition
```

**Code:**

```python
# Example code snippet for state handoff using Apache Spark
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("StateHandoff").getOrCreate()

# Define the source and target shards
source_shards = [shard1, shard2]
target_shards = [new_shard1, new_shard2]

# Transfer state using batch processing
for old_shard in source_shards:
    for new_shard in target_shards:
        # Perform state transfer logic here
        pass

# Example code snippet for incremental backfill using Apache Spark Streaming
from pyspark.streaming.kafka import KafkaUtils

kafka_stream = KafkaUtils.createDirectStream(
    ssc,
    ["topic_name"],
    {"metadata.broker.list": "localhost:9092"}
)

# Process data from the stream
def process_data(message):
    # Process each message and update state accordingly
    pass

kafka_stream.foreachRDD(process_data)
```

**Example input / output:**

| Old Shards | New Shards |
|------------|------------|
| shard1     | new_shard1 |
| shard2     | new_shard2 |
| shard3     | new_shard3 |
| shard4     | new_shard4 |

**Data:**

```json
{
  "source_shards": ["shard1", "shard2", "shard3", "shard4"],
  "target_shards": ["new_shard1", "new_shard2", "new_shard3", "new_shard4"]
}
```

**Flow:**

1. +-------------------------------------------------------+
2. | +---------------------+                             |
3. | | Source Shard Setup  |<--+                         |
4. | +---------------------+   | State Transfer          |
5. |                          v                         |
6. | +---------------------+  +-------------------------+
7. | | Monitoring         |<--| Incremental Backfill    |
8. | +---------------------+  +-------------------------+
9. |                          |                         |
10. |                          v                         |
11. | +---------------------+                             |
12. | | Cutover Plan        |->> Success                   |
13. | +---------------------+                             |
14. +-------------------------------------------------------+

