---
type: "practice"
difficulty: "Deep"
---

# A real-time feature counts events in a 5-minute window and is served at inference, but a late or duplicate event corrupts the count. Explain the windowing and exactly-once-versus-at-least-once concern and how event-time watermarking keeps the served feature correct.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Clock Skew and Time Windows](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are dealing with a real-time system where events are being ingested and processed in 5-minute windows to compute feature counts that will be used at inference time. The challenge is ensuring that late or duplicate events do not corrupt the count. To address this, we need to understand the concepts of windowing, exactly-once processing, and event-time watermarking.

First, let's break down the problem: Events are incoming in real-time, but they could be out-of-order (late) or even duplicated. The goal is to maintain an accurate 5-minute count despite these challenges. To solve this, we need to leverage a combination of windowing techniques and exactly-once processing guarantees.

#### Triaging the Question

The core concern here is ensuring that the real-time feature counts are correct in the presence of late events and duplicates. The interviewer might be probing your understanding of:

Windowing Techniques: How to group incoming events into fixed-size windows.
Exactly-once Processing: Ensuring each event is processed exactly once, avoiding duplication or loss.
Event-Time Watermarking: A mechanism to handle out-of-order events by setting a watermark that signals when late events are no longer expected.

#### Assumptions
and First Principles
Late Events: Events that arrive after the window has closed but before an explicit timeout.
Duplicate Events: The same event arriving multiple times.
Watermarking: A mechanism to mark the boundary of known-event-time, ensuring late events are handled appropriately.
Detailed Solution
1. Windowing

Windowing is a technique used in stream processing frameworks like Apache Flink or Kafka Streams to group incoming data into fixed-sized intervals (e.g., 5-minute windows). Each window accumulates events and processes them as a batch at the end of the window period.

For example, if an event arrives at time ( t ), it will be placed in the current window that ends at ( t + 5 ) minutes. The processing logic is applied once per window, meaning each event is processed exactly once within its respective window.

2. Exactly-once Processing

Exactly-once processing ensures that every event is processed precisely once, regardless of whether it arrives multiple times or with some out-of-order events. This can be achieved using transactional mechanisms and idempotent operations. For instance:

Transaction Mechanism: Each event is part of a transaction that starts when the event is ingested and ends when the processing completes.
Idempotency: Operations are designed to handle duplicate invocations without changing the outcome.
3. Event-Time Watermarking

Event-time watermarking is crucial for handling late events gracefully. The concept involves setting a boundary (watermark) that marks the latest event time seen so far, effectively "cutting off" further late events. Here’s how it works:

Watermark Calculation: Initially, the system sets a low watermark at the start of processing.
Watermark Advancement: As new events arrive and are processed, the watermark advances to reflect the latest known event time.
Late Event Handling: When an event arrives after the current watermark, it is considered late. The system can choose to either process it or discard it based on its configuration.
Putting It All Together

Here’s a step-by-step approach:

Window Creation: Events are ingested and placed into 5-minute windows.
Watermark Initialization: A low watermark is set at the start.
Event Processing:
Each event triggers a processing operation within its respective window.
The event time is checked against the current watermark to determine if it’s late.
Window Closure and Processing:
Once all events in a window have been processed, the window closes, and the feature count is computed.
The watermark advances to reflect the latest known event time.

#### Closing Summary

Event-time watermarking ensures that late or duplicate events do not corrupt the 5-minute window counts by setting boundaries on when late events are expected. This technique, combined with exactly-once processing guarantees, maintains the accuracy of real-time feature counts in a distributed system.

#### Common Mistakes
and Follow-ups
Ignoring Late Events: Forgetting to handle late events can lead to incorrect counts.
Overly Complex Watermarking Logic: Overcomplicating watermark advancement logic might introduce bugs or inefficiencies.
Inadequate Error Handling: Failing to account for duplicate events can cause processing anomalies.

**Comparison:**

```
Time	Event ID	Timestamp	Watermark
90	-	-	95
95	A	100	100
100	B	200	200
350	C	350	350
```

**Code:**

```java
// Example of a simple watermark calculation in Apache Flink
public class EventProcessor {
    private long watermark = 0; // Initial low watermark

    public void processEvent(Event event) {
        if (event.timestamp > watermark) {
            watermark = event.timestamp;
            // Advance the watermark based on the event timestamp
        }
        // Process the event
    }

    public void closeWindow() {
        // Compute and return the feature count for the window
    }
}
```

**Example input / output:**

```text
Input:
- Event A at time 95
- Event B at time 100 (in the same window)
- Event C at time 350 (late event)

Output:
- Window ends, feature count computed based on {A, B}
- Watermark advances to 350, late event ignored in this window.
```

**Data:**

```json
{
  "events": [
    {"id": 1, "timestamp": 100},
    {"id": 2, "timestamp": 150},
    {"id": 3, "timestamp": 200},
    {"id": 4, "timestamp": 250}
  ]
}
```

**Flow:**

1. Timestamp<95
2. Watermark=95
3. Timestamp>100
4. Yes
5. No
6. Event A
7. Skip
8. Watermark=100
9. Event B
10. Process
11. Is Watermark > 350?
12. Ignore Late Event
13. Process and AdvanceWatermark

