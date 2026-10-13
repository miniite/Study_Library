---
type: "practice"
difficulty: "Intermediate"
---

# If a streaming job goes down for an hour, what happens to the data during that gap and how could you recover it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Batch and Stream Reconciliation](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Batch_and_Stream_Reconciliation/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a streaming job encounters downtime, any data that arrives during this period is not processed until the system comes back online. This can lead to data latency and potentially loss if the processing window is not designed to handle such interruptions gracefully. To recover from such gaps, we need to implement mechanisms for replaying missed events or ensuring they are stored in a fault-tolerant manner.

This question delves into understanding the differences between batch and stream processing, specifically focusing on how failures impact data continuity. The key concepts include event storage durability, state management, and error handling strategies. We will explore these aspects step-by-step to build a robust solution.

**Comparison:**

```
Event Timestamp	Data Type	User ID
2023-10-01T10:00Z	user-login	123
2023-10-01T10:01Z	user-login	456
```

**Code:**

```python
# Example of code snippet for replaying missed events in a streaming job using Apache Flink
def process_event(event):
    # Process the event normally here
    pass

def handle_replay(event):
    if not event_processed_flag[event]:
        process_event(event)
        event_processed_flag[event] = True
```

**Example input / output:**

```text
Input:
{
  "events": [
    {"timestamp": "2023-10-01T10:00Z", "data": {"type": "user-login", "details": {"user_id": 123}}},
    {"timestamp": "2023-10-01T10:01Z", "data": {"type": "user-login", "details": {"user_id": 456}}}
  ]
}

Output:
Processed events:
- User login at 10:00
- No processing for the event at 10:01 due to downtime

Recovery after downtime:
- After reboot, replay missed event at 10:01Z
```

**Data:**

```json
{
  "events": [
    {"timestamp": "2023-10-01T10:00Z", "data": {"type": "user-login", "details": {"user_id": 123}}},
    {"timestamp": "2023-10-01T10:01Z", "data": {"type": "user-login", "details": {"user_id": 456}}}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Event Arrival
5. Is System Up?
6. Process Event
7. Store Event for Replay
8. Recovery After Downtime
9. Are There Missed Events?
10. Replay Missed Events

