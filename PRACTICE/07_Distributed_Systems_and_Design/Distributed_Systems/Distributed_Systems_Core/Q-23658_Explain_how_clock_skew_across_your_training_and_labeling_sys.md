---
type: "practice"
difficulty: "Deep"
---

# Explain how clock skew across your training and labeling systems can corrupt a time-windowed training set (e.g. features from 'before' the label actually being after). How would you defend against it with event-time rather than processing-time semantics?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clock Skew and Time Windows](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Clock skew in distributed systems, particularly when dealing with time-windowed training sets like those used in machine learning, can introduce significant issues such as label corruption if not properly managed. This is because clock differences between nodes or systems can lead to events being processed at different times, potentially placing them into incorrect windows. The goal of defending against this issue involves using event-time semantics over processing-time semantics.

Firstly, let’s clarify what we mean by these terms:

Processing-Time Semantics: Events are assigned timestamps based on when they were processed.
Event-Time Semantics: Events retain their own timestamps, which represent the actual time they occurred in the real world.

By using event-time semantics, we ensure that each piece of data retains its original timestamp, thereby reducing the risk of corruption due to clock skew. This approach is crucial for maintaining temporal consistency across distributed systems.

To illustrate this concept further, consider a scenario where you are training a model on customer interactions. Each interaction has an associated timestamp indicating when it occurred in reality. If we use processing-time semantics, events might be processed and assigned timestamps based on the server’s local clock. Clock skew could mean that some events recorded as happening at 9:00 AM might actually have been recorded at 8:59 AM or 9:01 AM by different nodes.

Here's a step-by-step explanation of how to defend against this issue:

Step-by-Step Solution

#### Triage

The key challenge is ensuring that events are correctly placed in their intended time windows based on their original timestamps. This involves:

Maintaining accurate and synchronized clocks across all systems.
Using event-time semantics to preserve the actual timing information.
Implementing strategies for clock synchronization.
First Principles
Event-Time Semantics: Each piece of data has an associated timestamp that represents the time it occurred in reality.
Clock Synchronization: Ensure all nodes have synchronized clocks using protocols like NTP (Network Time Protocol) or PTP (Precision Time Protocol).
Watermarking: Introduce watermark signals to mark the end of a known event window, ensuring late events are dropped rather than processed incorrectly.
Detailed Solution

Timestamps and Watermarks:

Collect data with accurate timestamps.
Implement watermark generation to mark the end of a time window.
Process events up to the current watermark, discarding any that arrive after it.

Synchronized Clocks:

Use NTP or PTP to synchronize clocks across all nodes.
Monitor and adjust clock drift regularly to maintain accuracy.

Event Processing Pipeline:

Design a streaming pipeline where each event includes its timestamp.
Use a windowing mechanism that processes events based on their timestamps rather than the time they were processed.

Failure Mode Handling:

Implement robust error handling and logging for out-of-order events.
Use backpressure mechanisms to prevent overloading when dealing with clock skew.

#### Example Scenario

#### Cons
ider a real-time anomaly detection system where user interactions are tracked in real-time. Each interaction has an associated timestamp:

User A logs in at 10:00 AM (Node 1).
User B performs an action at 10:05 AM (Node 2).

If Node 2's clock is slow, the actual event might be recorded as 10:03 AM. When processed by a central system using processing-time semantics, it might end up in the wrong time window.

By instead using event-time semantics and accurate clocks:

The system will recognize that User B’s action was truly at 10:05 AM.
Events are placed correctly into their intended windows, maintaining temporal consistency.

#### Closing Summary

Using event-time semantics and robust clock synchronization mechanisms is essential to defending against clock skew in distributed training systems. This approach ensures that data retains its original timing information, reducing the risk of corruption due to differences in local clocks.

#### Common Mistakes/Follow-Ups
Forgetting to Synchronize Clocks: Ensuring all nodes have synchronized clocks is crucial but often overlooked.
Incorrect Watermarking Strategy: Poorly implemented watermarking can lead to false positives or negatives, affecting model accuracy.
Handling Out-of-Order Events: Failing to implement proper backpressure mechanisms can overwhelm the system with late events.

**Comparison:**

```
NTP Sync
PTP Sync
Watermark Generation
Event Processing
Backpressure Implementation
Clock Synchronization
Event Processing Pipeline
Failure Mode Handling
Event Time vs Processing Time
```

**Code:**

```python
# Example of a simple event-time processing pipeline in Python

import time
from datetime import datetime

class Event:
    def __init__(self, timestamp):
        self.timestamp = timestamp

def process_events(events, watermark):
    current_time = datetime.now().timestamp()

    for event in events:
        if event.timestamp <= current_time and event.timestamp < watermark:
            print(f"Processing {event.timestamp}")
        else:
            print(f"Dropping late event: {event.timestamp} > {current_time}")

# Simulating events with timestamps
events = [Event(time.mktime(datetime(2023, 10, 5, 10, 4, 0).timetuple())),
          Event(time.mktime(datetime(2023, 10, 5, 10, 5, 0).timetuple())),
          Event(time.mktime(datetime(2023, 10, 5, 10, 6, 0).timetuple()))]

# Simulating watermark
watermark = time.mktime(datetime(2023, 10, 5, 10, 4, 30).timetuple())

process_events(events, watermark)
```

**Example input / output:**

| Event | Watermark | Processed? |
|-------|-----------|------------|
| [1696474800.0] | 1696474830.0 | Yes         |
| [1696475100.0] | 1696474830.0 | No          |
| [1696475400.0] | 1696474830.0 | No          |

**Data:**

```json
{
    "events": [
        {"timestamp": 1696474800.0},
        {"timestamp": 1696475100.0},
        {"timestamp": 1696475400.0}
    ],
    "watermark": 1696474830.0
}
```

**Flow:**

1. Yes
2. No
3. Receive Event
4. Is Time > Watermark?
5. Drop Late Event
6. Process Event
7. Return to Ready State
8. Log Processed Event

