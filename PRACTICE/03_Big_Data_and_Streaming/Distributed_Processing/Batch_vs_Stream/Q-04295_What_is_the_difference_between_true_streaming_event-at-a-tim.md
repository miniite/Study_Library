---
type: "practice"
difficulty: "Deep"
---

# What is the difference between true streaming (event-at-a-time) and micro-batch streaming?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Batch vs Streaming Cost Trade-offs](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_vs_Streaming_Cost_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the core differences between true streaming and micro-batch streaming, let's start with a brief overview. True streaming processes data as it arrives in real time, treating each event as an independent unit. Micro-batch streaming, on the other hand, groups events into small batches before processing them together. The key distinction lies in how these systems handle data ingestion and processing.

The question is really probing your understanding of the fundamental differences between true streaming and micro-batch streaming approaches, their trade-offs, and the contexts in which each might be more suitable. Let's break it down step by step from first principles to provide a comprehensive solution.

**Comparison:**

| True Streaming       | Micro-Batch Streaming                |
|----------------------|--------------------------------------|
| Event-at-a-time      | Batch of events                      |
| Real-time processing | Periodic batch processing            |
| Low latency          | Higher latency (due to batching)     |
| High scalability     | Lower scalability                    |

**Example input / output:**

| True Streaming       | Micro-Batch Streaming                |
|----------------------|--------------------------------------|
| Event-at-a-time      | Batch of events                      |
| Real-time processing | Periodic batch processing            |
| Low latency          | Higher latency (due to batching)     |
| High scalability     | Lower scalability                    |

**Data:**

```json
{
  "true_streaming": {
    "description": "Processes each event as an independent unit in real time."
  },
  "micro_batch_streaming": {
    "description": "Groups events into small batches before processing them together."
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. True Streaming
6. Is the data arrivalevent-at-a-time?
7. Process each event in real time
8. Micro-Batch Streaming
9. Is the batch size small andfrequent?
10. Periodic processing, but withlow latency
11. Large batches, lowerfrequency, higher latency

