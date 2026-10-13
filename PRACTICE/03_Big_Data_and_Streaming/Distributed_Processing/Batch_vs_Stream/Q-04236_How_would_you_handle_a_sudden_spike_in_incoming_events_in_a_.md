---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle a sudden spike in incoming events in a streaming pipeline?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Correctness and Drift](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Streaming_Correctness_and_Drift.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a streaming pipeline, handling a sudden spike in incoming events is crucial to maintain system stability and ensure data integrity. Batch processing systems can struggle with such spikes due to their batch-oriented nature, while stream processing systems are more adept at dealing with variable workloads. In this scenario, we need to focus on maintaining the throughput and latency of our pipeline during high event volumes.

To handle a sudden spike in incoming events, let’s break down the problem from first principles and consider various aspects such as buffer management, backpressure mechanisms, and stateful processing strategies.

**Comparison:**

```
Metric	Batch Processing	Stream Processing
Latency	High (batch)	Low
Throughput	Variable (batch)	Consistent
Scalability	Poor (batch)	Good (stream)
```

**Flow:**

1. Detect Spike
2. Buffer Management
3. Apply Backpressure
4. Process Events
5. Maintain Consistency
6. Notify/Alert Team

