---
type: "practice"
difficulty: "Deep"
---

# How do you ensure a streaming job does not lose data when it restarts after a crash?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Batch and Stream Reconciliation](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a streaming job does not lose data when it restarts after a crash, we need to focus on the design and implementation of state management mechanisms that can handle failures gracefully. At a high level, this involves understanding the nature of stream processing, the guarantees provided by different frameworks, and implementing robust recovery strategies.

#### Triaging the Question

The core of the question is about ensuring data integrity in stream processing systems when there's an interruption due to crashes or other issues. The interviewer will be probing your knowledge of fault tolerance mechanisms, state management techniques, and best practices for designing resilient streaming applications.

#### Assumptions
Framework: We are using a typical stream processing framework like Apache Kafka Streams, Flink, or Spark Streaming.
State Management: State is maintained across the lifetime of the application to ensure consistency.
Crash Scenario: The job can crash at any point in time during execution.

#### First-Principles Solution

Event Time vs Processing Time:

Ensure that your processing logic is based on event-time rather than processing-time for stateful operations, as this allows the system to maintain a consistent view of events even if there are delays or out-of-order messages.

State Stores and Checkpoints:

Implement state stores in frameworks like Flink to persist state periodically through checkpointing. Checkpoints ensure that the application can be restored from a known good state after a failure.
Configure periodic checkpoints at intervals suitable for your use case, balancing between recovery time and resource consumption.

Exactly Once Semantics:

For critical operations, implement exactly-once semantics to avoid duplicate or dropped messages. This might involve using transactional messaging protocols if supported by the framework.

Fault Tolerance Strategies:

Use backpressure mechanisms to prevent overloading of processing nodes and ensure that the system can handle failures gracefully without losing data.

Graceful Restart Mechanisms:

Design your application to restart from a checkpoint rather than starting fresh, ensuring minimal state loss.
Implement logic to detect and recover from partial failures, such as reprocessing failed batches or messages.

#### Closing Summary

By carefully managing state through periodic checkpoints, using event-time processing, implementing exactly-once semantics where necessary, and designing for graceful restarts, we can ensure that a streaming job does not lose data when it crashes. Each of these strategies contributes to building a resilient system capable of handling interruptions effectively.

#### Common Mistakes/Follow-Ups
Not Using Checkpoints: Failing to configure periodic checkpoints can lead to significant state loss during restarts.
Over-reliance on Processing Time: Relying solely on processing time for stateful operations without considering event-time can cause inconsistencies and data skew.
Ignoring Backpressure: Not implementing backpressure mechanisms can result in the job being overwhelmed by incoming data, leading to potential failures.
Incomplete Recovery Logic: Failing to implement robust logic for detecting partial failures and handling them appropriately can leave gaps in data processing.

**Comparison:**

```
Operation	Description
Event Time Processing	Ensures consistent processing regardless of message order and latency.
Checkpoints	Periodically save the application state to enable recovery from a known good point.
Exactly Once Semantics	Prevents duplicate or dropped messages, ensuring data integrity.
```

**Code:**

```java
// Example of setting up periodic checkpointing in Apache Flink
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.enableCheckpointing(5000); // Checkpoint every 5 seconds

DataStream<MyEvent> events = env.addSource(new KafkaSource());
StateTtlConfig ttlConfig = StateTtlConfig
    .newBuilder(Time.minutes(60)) // Set TTL for state
    .setUpdateType(StateTtlConfig.UpdateType.OnCreateAndWrite)
    .build();
MapStateDescriptor<String, MyEvent> eventsStateDesc = new MapStateDescriptor<>(
    "events", Types.STRING(), Types.POJO(MyEvent.class));
```

**Example input / output:**

```text
Input:
- Event: {"event": {"id": "12345", "value": "Hello World", "timestamp": "2023-10-01T12:00:00Z"}}

Output (after checkpoint):
- Checkpointed State: {"12345": {"value": "Hello World"}}
```

**Data:**

```json
{
  "event": {
    "id": "12345",
    "value": "Hello World",
    "timestamp": "2023-10-01T12:00:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Streaming Job
4. Crashes
5. Is Checkpoint Enabled?
6. Restore State from Last KnownGood Checkpoint
7. Restart from Scratch withPotential Data Loss

