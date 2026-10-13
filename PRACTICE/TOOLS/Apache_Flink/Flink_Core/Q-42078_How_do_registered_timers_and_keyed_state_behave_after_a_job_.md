---
type: "practice"
difficulty: "Deep"
---

# How do registered timers and keyed state behave after a job restarts from a checkpoint, and why does that matter for session flushing?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Checkpointing and Recovery](<../../../../TOOLS/Apache_Flink/Flink_Core/Checkpointing_and_Recovery.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how registered timers and keyed state behave after a job restarts from a checkpoint in Apache Flink, let's first outline the core concepts involved and then delve into the specifics of their behavior during a job recovery process.

TL;DR

Registered timers are reset to their original timestamps upon job recovery, while keyed state persists across checkpoints. This difference is crucial for maintaining data integrity, especially in scenarios where session-based state management is required.

#### Triaging the Question

The question delves into the intricacies of stream processing with Apache Flink, specifically focusing on two key aspects:

Behavior of Timers: How timers are handled when a job restarts from a checkpoint.
Keyed State Behavior: What happens to keyed state during recovery.

#### Assumptions
The system uses Apache Flink for real-time stream processing.
Checkpoints have been enabled and are being used effectively.
We understand the basic lifecycle of a Flink job, including execution, failure, checkpointing, and recovery.

#### First Principles Solution

Checkpoints in Flink

Flink periodically saves snapshots (checkpoints) of the state, which includes both timers and keyed state.
Checkpointing ensures that if a job fails or needs to be restarted, it can recover from the latest checkpointed state.

Timers Behavior:

Timers in Flink are stored as part of the keyed state during checkpoints.
When a job restarts from a checkpoint, timers are reset to their original timestamps. This means that if a timer was set for a specific timestamp before failure, it will be reset upon recovery and any scheduled events will start over.

Keyed State Behavior:

Keyed state is designed to be consistent across checkpoints.
When a job restarts, keyed state is restored from the latest checkpoint, ensuring that all data associated with keys is in its last known good state before failure.

Session-Based State Management:

Sessions can often be thought of as groups of events that are processed together over time.
If session-based state management relies on timers (e.g., to define the boundaries of sessions), these sessions may not survive a job restart without careful design.
Why It Matters for Session Flushing
Session Flush: In many real-time applications, flushing sessions is crucial. For example, in log analysis or fraud detection systems, detecting events within a session window is essential.
Failure Recovery: If timers are reset and sessions start over from the beginning upon recovery, this can lead to incorrect conclusions about ongoing sessions if not properly managed.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Common Mistake: Assuming that timers will continue to run as they did before failure. This is a frequent oversight in designing fault-tolerant systems.

#### Follow-Up
Ensuring that session-based logic is designed to handle restarts gracefully, possibly by using custom recovery strategies or checkpoints with additional metadata.
Summary

Understanding the behavior of timers and keyed state during job recovery in Flink helps in designing robust stream processing applications. Timers reset on recovery, while keyed state remains consistent, which impacts how sessions are managed and flushed after failures.

**Comparison:**

```
Timer Behavior	Keyed State Behavior
Reset to origin	Restored from last
```

**Code:**

```java
// Example code snippet for starting a Flink job with checkpointing enabled
StreamExecutionEnvironment env = StreamExecutionEnvironment.getExecutionEnvironment();
env.enableCheckpointing(5000); // 5 seconds interval

DataStream<MyType> input = env.addSource(sourceFunction);

input
    .keyBy(keySelector)
    .process(MyProcessor.class)
    .print();

env.execute("Checkpointed Flink Job Example");
```

**Example input / output:**

```text
Input:
- Data events arriving in a stream with timestamps and keys.

Output after recovery:
- Timers are reset to their original timestamps.
- Keyed state is restored from the latest checkpoint, ensuring consistency.
```

**Data:**

```json
{
  "checkpointInterval": 5000,
  "minPauseBetweenCheckpoints": 2000,
  "enableExternalizedCheckpoints": true
}
```

**Flow:**

1. Checkpoint
2. Job Failure
3. Restore Checkpoint
4. Timers
5. Reset to Original
6. Keyed State
7. Restored from Last Checkpoint

