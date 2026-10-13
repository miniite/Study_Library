---
type: "practice"
difficulty: "Deep"
---

# Late-arriving log events routinely show up two minutes after their event-time in your observability pipeline. How do you set allowed-lateness and watermarks so your rolling error-rate window doesn't fire prematurely and page on-call for nothing?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event-time Watermarks](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address late-arriving log events that typically arrive two minutes after their event-time, we need to configure our streaming architecture to ensure robustness without overreacting to false positives. This involves setting appropriate allowed-lateness and watermarks in a way that balances accuracy with system performance.

#### Triaging the Question

The core of this problem is understanding how late-arriving data impacts state estimation and error detection in real-time systems, specifically within the context of observability pipelines. The interviewer is probing your knowledge of windowing mechanisms, watermarking strategies, and their trade-offs in streaming architectures.

#### First Principles Solution
Understand Late-Arriving Data: In a stream processing pipeline, late-arriving data can cause issues with state updates and error detection windows.
Set Allowed-Lateness: This parameter defines how long after the event-time we allow data to arrive without it being considered late. A two-minute delay is common in many systems.
Watermark Mechanism: Watermarks are used to mark the latest known event-time, ensuring that any data before this timestamp is considered "late" if not processed.
Step-by-Step Solution

Define Allowed-Lateness:

Given the late-arriving logs come in two minutes after their event-time, we need to set allowed-lateness to at least 2 minutes.
Yes
No
Event-Time
Late-Arrival?
Wait for Allowed-Lateness
Process Event
Process Event

Implement Watermarks:

Watermarks should be generated based on the latest known event-time. If a watermark is set at 10:00 AM, any data older than this timestamp will be considered late and not processed until the next watermark.

Error Window Configuration:

The error window determines how long state updates are valid before they expire. This should account for both the allowed-lateness and typical event processing time.

#### Example
Configuration
Watermark: Set to 10:00 AM, or any timestamp that is known to be accurate.
Allowed-Lateness: 2 minutes (i.e., data can arrive up to 2 minutes after its event-time).
Error Window: Should span the allowed-lateness plus an additional buffer time. For example, if state updates are valid for 5 minutes, then the error window should be set to approximately 7 minutes.

#### Closing Summary

By setting the allowed-lateness and watermarks correctly, we can ensure that our system handles late-arriving data gracefully without generating false positives or causing unnecessary paging of on-call personnel. The key is to balance accuracy with performance by allowing for a reasonable amount of lateness while ensuring timely error detection.

#### Common Mistakes/Follow-Ups
Ignoring Lateness: Not accounting for late-arriving data can lead to frequent false alarms.
Overly Aggressive Windowing: Setting the allowed-lateness too low can result in missed events or increased paging frequency.
Lack of Buffer Time: Insufficient buffer time in error windows can cause state updates to expire prematurely.

To further refine your approach, consider implementing robust watermark generation strategies and fine-tuning the error window based on real-world data patterns.

**Comparison:**

```
Parameter	Value
Allowed-Lateness	2 minutes
Watermark	10:00 AM
Error Window	7 minutes
```

**Code:**

```python
# Example code snippet for setting allowed-lateness and watermarks
allowed_lateness = 120  # 2 minutes in seconds
watermark_timestamp = "2023-10-05T10:00:00Z"  # Set to latest known event-time

# In a streaming pipeline configuration
pipeline_config = {
    "allowed_lateness": allowed_lateness,
    "watermark": watermark_timestamp,
    "error_window_duration": 360,  # 6 minutes in seconds for example
}
```

**Example input / output:**

```text
Input:
Event-Time: 2023-10-05T10:00:01Z

Watermark Check:
- Current Time: 2023-10-05T10:02:00Z (2 minutes after event-time)
- Watermark Timestamp: 2023-10-05T10:00:00Z

Output:
Event processed normally
```

**Data:**

```json
{
  "allowedLateness": 120,  // 2 minutes
  "watermarkTimestamp": "2023-10-05T10:00:00Z",
  "errorWindowDuration": 420  // 7 minutes to account for allowed lateness and buffer time
}
```

**Flow:**

1. Yes
2. No
3. Event-Time
4. Late-Arrival?
5. Wait for Allowed-Lateness
6. Process Event
7. Process Event

