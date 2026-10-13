---
type: "practice"
difficulty: "Deep"
---

# How do you handle out-of-order events at scale so both the streaming aggregate and the batch recompute converge to the same value, and how long is your "grace period" org-wide?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Unified Batch and Streaming Semantics](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/Unified_Batch_and_Streaming_Semantics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Handling out-of-order events in a way that ensures both streaming aggregates and batch recomputes converge to the same value requires a deep understanding of event time processing, watermarking strategies, and robust state management techniques. The "grace period" or latency tolerance is a critical parameter that defines how long our system can tolerate delays before it starts to diverge from real-time accuracy.

#### Triage

The question essentially probes your knowledge on managing event order in real-time systems, ensuring consistency between batch and streaming data processing, and defining the acceptable latency. Key assumptions include:

The system handles both historical and current events.
There is a need for near-real-time (NRT) results as well as periodic full recompute.

#### First-Principles Solution
Event Time Processing: Convert all timestamps to event time to ensure correct chronological ordering of events, regardless of their arrival order.
Watermarking: Introduce watermarking to mark when the latest known data timestamp is for a stream. This helps in discarding late arriving events that do not contribute to the current processing window.
State Management: Use stateful processing with key-value stores or distributed state machines like Kafka Streams, Flink State Backends, etc., to maintain consistent state across both streaming and batch computations.
Grace Period Definition: Define a global "grace period" that accounts for network latency, system delays, and data skew.
Detailed Solution

Event Time Processing:

Convert all events to have an event timestamp rather than processing time.
This ensures that the order of events is preserved irrespective of when they are processed.

Watermarking:

Implement a watermark generator that defines the latest known event timestamp for each key in the stream.
Use this watermark to determine if incoming late events should be ignored or included in processing.

State Management:

Utilize stateful processing frameworks like Apache Flink, Apache Spark Streaming, or Kafka Streams to maintain consistent state across both streaming and batch computations.
Ensure that state is replicated and fault-tolerant to handle failures gracefully.

Grace Period Definition:

Define a global grace period ( G ) in seconds, which represents the maximum allowable delay before events are considered late.
This value should be tuned based on network conditions, system performance, and business requirements.
Grace Period Calculation

The "grace period" can be calculated using the following steps:

Network Latency: Estimate the average latency between different nodes in your distributed architecture.
System Processing Time: Measure the time taken for each processing window to complete.
Data Skew: Account for any potential skew in data arrival times, especially if there are periodic spikes in event generation.

For example, if network latency is 100 ms and system processing takes 5 seconds with some variation due to skew, a grace period of around 6-7 seconds might be reasonable. However, this needs to be validated through monitoring and performance testing.

#### Closing Summary

In summary, handling out-of-order events at scale involves leveraging event time processing, watermarking strategies, robust state management, and defining an appropriate "grace period" to ensure consistency between streaming and batch computations. The grace period should be tuned based on network conditions, system performance, and business requirements to minimize delays while maintaining accuracy.

#### Common Mistakes/Follow-Ups
Ignoring Late Events: Failing to handle late events properly can lead to state inconsistencies.
Overly Aggressive Grace Periods: Setting the grace period too short might result in discarding useful late data.
Insufficient State Management: Poor state management can lead to race conditions and loss of consistency.

Would you like me to elaborate on any specific aspect, such as architectural choices or performance trade-offs?

**Comparison:**

```
Grace Period	Network Latency	Processing Time
60 seconds	100 ms	5 seconds
```

**Code:**

```python
# Example watermarking implementation using Flink
from pyflink.datastream import StreamExecutionEnvironment
from pyflink.table import StreamTableEnvironment

env = StreamExecutionEnvironment.get_execution_environment()
t_env = StreamTableEnvironment.create(env)

# Define a watermark strategy with a grace period of 60 seconds
watermark_strategy = WatermarkStrategy.for_monotonous_timestamps().with_grace_period(Duration.ofSeconds(60))

# Register the table environment and apply watermarking to the stream
t_env.register_table_source(source)
t_env.register_table_sink(sink)

table = t_env.from_path("source")
table.with_row_time_attributes(table.select(timestamp_field).rowtime(watermark_strategy))
```

**Example input / output:**

```text
Input:
[{"id": 1, "value": 10, "timestamp": "2023-10-01T09:00:00Z"},
 {"id": 2, "value": 20, "timestamp": "2023-10-01T09:05:00Z"},
 {"id": 3, "value": 30, "timestamp": "2023-10-01T08:55:00Z"}, // Late event
 {"id": 4, "value": 40, "timestamp": "2023-10-01T09:10:00Z"}]

Watermark Strategy:
Grace Period = 60 seconds

Output (assuming a processing window of 5 minutes):
Aggregate Value: 10 + 20 + 30 = 60
Late Event Handling: Ignored due to grace period expiration
```

**Data:**

```json
{
  "events": [
    {"id": 1, "value": 10, "timestamp": "2023-10-01T09:00:00Z"},
    {"id": 2, "value": 20, "timestamp": "2023-10-01T09:05:00Z"},
    {"id": 3, "value": 30, "timestamp": "2023-10-01T08:55:00Z"}, // Late event
    {"id": 4, "value": 40, "timestamp": "2023-10-01T09:10:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Event
4. Is Late?
5. Ignored
6. Processed in Current Window
7. Watermark Update
8. State Update & Aggregation

