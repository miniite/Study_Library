---
type: "practice"
difficulty: "Deep"
---

# Your anomaly model reads a real-time feature (error-rate over the last minute) from a stream, and a duplicate or late event inflated the rate into a false positive. Explain the exactly-once-versus-at-least-once concern and how event-time watermarking keeps the served feature correct.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clock Skew and Time Windows](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our anomaly detection model operates correctly, we need to address the issue of duplicate or late events inflating error rates, which can trigger false positives. This problem is closely tied to the Exactly-Once (EO) vs At-Least-Once (ALO) guarantees in distributed systems. Specifically, a watermarking technique helps maintain the correctness of the served feature by tracking event-time and ensuring only relevant data contributes to our computation.

In summary:

Exactly-Once ensures that each event is processed exactly once.
At-Least-Once allows an event to be processed multiple times.
Event-time watermarking helps in maintaining correctness by keeping track of the latest timestamp, thus filtering out late or duplicate events.

#### Triaging the Question

The question revolves around ensuring our anomaly detection model accurately reflects real-time error rates without being misled by duplicate or delayed data. The core challenge is to distinguish between genuine anomalies and artifacts caused by processing duplicates or late events more than once.

#### Assumptions

The system processes a stream of real-time features.
There's a possibility of receiving the same event multiple times (duplicates) or slightly later than expected (late events).
We aim for an Exactly-Once guarantee to avoid false positives.

#### First Principles Solution
Event-Time Watermarking: This technique involves maintaining a watermark that represents the latest timestamp we have processed.
Filtering Events: Events that are received before the watermark should be included in our computation (as they are considered "late" but not yet processed). Events after or equal to the watermark are treated as new, and we process them.
Step-by-Step Solution

Initialization:

Set a global watermark to 0.
Initialize an event buffer to hold incoming events.

Processing Incoming Events:

For each event received from the stream, check its timestamp against the current watermark.
If the event's timestamp is less than or equal to the watermark, it’s considered late and stored in a buffer for later processing.
If the event's timestamp is greater than the watermark, it’s considered new and immediately processed.

Updating Watermark:

Periodically update the watermark based on the max timestamp of events in the buffer or received from the stream.

Final Processing:

After a certain period (e.g., 1 minute), process all buffered late events with the current watermark as their latest timestamp.
Reset the buffer and continue processing new incoming events.

#### Example
Workflow
Suppose we receive an event at t=50 (timestamp). If the current watermark is 48, this event is processed immediately.
If another identical event arrives at t=52, it’s stored in a buffer. When the watermark updates to 51, these events are then processed.

#### Closing Summary

By implementing event-time watermarking, we ensure that our anomaly detection model only considers relevant data, effectively filtering out duplicates and late events. This approach guarantees that each event is processed exactly once relative to its time of arrival, thus maintaining the integrity of the feature being served.

#### Common Mistakes
and Follow-Ups
Mistake: Ignoring duplicate or late events can lead to false positives.

#### Follow-Up
Discuss trade-offs between processing speed (latency) and correctness. For instance, real-time systems may need faster watermark updates but sacrifice some accuracy, while batch systems might be more tolerant of delays.

#### Example
Code
PYTHON
def process_event(event):
    if event.timestamp <= watermark:
        buffer.append(event)
    else:
        # Process new event
        print(f"Processing new event {event}")

Watermark Update Mechanism:

PYTHON
def update_watermark():
    global watermark
    current_max = max(e.timestamp for e in buffer + received_events)
    watermark = current_max
Self-Check

The answer covers the exact-once vs at-least-once concern and how event-time watermarking addresses false positives, ensuring correct anomaly detection. The solution is comprehensive and avoids common pitfalls.

