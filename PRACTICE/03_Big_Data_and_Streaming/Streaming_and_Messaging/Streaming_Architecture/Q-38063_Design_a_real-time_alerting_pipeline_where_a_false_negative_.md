---
type: "practice"
difficulty: "Expert"
---

# Design a real-time alerting pipeline where a false negative (missed alert) is catastrophic but false positives erode trust. How do watermarks, allowed-lateness, and window firing policy interact with this asymmetry, and what defaults do you pick?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Event-time Watermarks](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Event-time_Watermarks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a real-time alerting pipeline where missing an alert (false negative) is catastrophic but false positives are less critical, we must carefully balance our watermark strategy, allowed-lateness, and window firing policy to minimize missed alerts while managing the cost of false positives. The key concepts here revolve around understanding how these elements interact with the asymmetry in the error costs.

#### Triage
Understanding the Question

The question is asking about designing a robust alerting system for critical events where an undetected event has severe consequences. We need to ensure that we capture as many true positives as possible while being mindful of the false positive rate, which can erode user trust over time if not controlled.

Triaging Assumptions
Event Criticality: The events are critical and have catastrophic outcomes.
Data Latency: Events may arrive with some delay due to network or processing latencies.
System Scale: We're dealing with a large-scale, high-velocity system where the volume of data can be massive.

#### First-Principles Solution

To address this scenario, we need an alerting pipeline that can handle real-time data streams efficiently while ensuring minimal latency and false negatives. Here’s how watermarks, allowed-lateness, and window firing policy interact:

Watermarks

Watermarks are used to indicate the latest time at which data has been processed for a given window. They help in managing late-arriving data by allowing some leeway (allowed-lateness) without triggering immediate alerts.

Watermark Concept: A watermark is essentially a timestamp that indicates when the system believes all relevant events up to a certain point in time have arrived.
Allowed Lateness: This parameter allows the processing of late-arriving data, but only for a limited period after the expected event window has closed.
Allowed-Lateness

Allowed-lateness is critical because it defines how long late events are allowed before they trigger an alert. For catastrophic false negatives, we need to ensure that any relevant data can still be processed within this time frame.

Choosing Allowed Lateness: Since missing a true positive event could have severe consequences, we set the allowed-lateness to a very small value (e.g., 5-10 seconds) to minimize the risk of false negatives.
Watermark Strategy: The watermark will be advanced only when all expected data has been received within the allowed lateness period.
Window Firing Policy

The window firing policy determines how and when alerts are generated based on the events processed in a given time window.

Event-Based Windows: We use event-based windows, where each batch of events is considered separately.
Firing Policy: Alerts should be fired immediately upon detecting an issue but only if the watermark confirms that all relevant data has been processed within the allowed lateness period.
Full Solution

Given these considerations, here’s a high-level design:

Event Processing Pipeline:

Each event is timestamped and routed to the appropriate processing window.
A watermark is generated for each window to indicate when the expected events should have arrived.

Late Data Handling:

Late-arriving data (up to allowed lateness) can still be processed but must be marked with a special tag indicating its late nature.

Window Firing and Alerting:

For each window, once the watermark confirms that all expected events have arrived within the allowed lateness period:
Process the events.
If any event triggers an alert condition (e.g., threshold exceeded), generate an immediate alert.

Notification Mechanism:

Ensure alerts are sent via multiple channels for redundancy (email, SMS, in-app notifications).

Monitoring and Feedback Loops:

Continuously monitor the system’s performance and tweak parameters like allowed lateness based on real-world data.

#### Closing Summary

By carefully setting the watermark policy to ensure minimal false negatives and using a small but reasonable allowed-lateness value, we can build an alerting pipeline that minimizes catastrophic consequences while keeping false positives in check. The key is to balance these elements to maintain both system reliability and user trust.

#### Common Mistakes/Follow-Ups
Ignoring Allowed Lateness: Overlooking the importance of late data handling can lead to missed alerts.
Inadequate Watermark Management: Improper watermark management can cause false positives or false negatives.
Over-Reliance on Thresholds: Relying solely on static thresholds without adaptability can fail under varying conditions.
Triaging

This solution ensures a robust alerting system that minimizes catastrophic errors while managing the impact of false positives effectively.

**Comparison:**

| Window | Allowed Lateness (s) | Events Processed | Watermark Generated At |
|--------|---------------------|------------------|-----------------------|
| 0-60   | 10                  | {"timestamp": "2023-10-01T00:59:45Z", "value": 1}, {"timestamp": "2023-10-01T01:00:05Z", "value": 1} | 2023-10-01T01:00:10Z |

**Code:**

```python
# Example Python pseudo-code for watermark and allowed-lateness handling

def process_event(event, window_start_time):
    # Process event based on current state
    if is_catastrophic_event(event):
        raise Alert("Catastrophic Event Detected")

def generate_watermark(window_end_time, allowed_lateness_seconds=10):
    return window_end_time + allowed_lateness_seconds

def check_window(window_events, watermark_timestamp):
    for event in window_events:
        if event.timestamp < watermark_timestamp:
            process_event(event)
```

**Example input / output:**

```text
Input:
Window start time: 2023-10-01T00:00:00Z, end time: 2023-10-01T01:00:00Z, allowed lateness: 10 seconds

Events processed in window:
[{"timestamp": "2023-10-01T00:59:45Z", "value": 1}, {"timestamp": "2023-10-01T01:00:05Z", "value": 1}]

Watermark generated at: 2023-10-01T01:00:10Z

Output:
No alert, as all events arrived within allowed lateness.
```

**Data:**

```json
{
  "window": {
    "start_time": "2023-10-01T00:00:00Z",
    "end_time": "2023-10-01T01:00:00Z",
    "allowed_lateness_seconds": 10
  },
  "events": [
    {"timestamp": "2023-10-01T00:59:45Z", "value": 1},
    {"timestamp": "2023-10-01T01:00:05Z", "value": 1}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Event Received
6. Is Event Time Within AllowedLateness?
7. Alert Generated
8. Wait for Watermark
9. Process Event and GenerateAlert
10. Watermark Confirmed
11. Process Events, CheckConditions
12. Condition Met?
13. Generate Immediate Alert
14. Continue Processing

