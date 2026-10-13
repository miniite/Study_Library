---
type: "practice"
difficulty: "Deep"
---

# Explain how clock skew across your event producers can corrupt a time-windowed aggregation so events that are 'before' actually land after. How do you enforce event-time semantics and monitor drift as a correctness SLI?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Clock Skew and Time Windows](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Clock skew among distributed systems can lead to significant issues in time-windowed aggregations, particularly when events are timestamped by producers rather than ingested timestamps. This scenario is common in real-time data pipelines where events from multiple sources need to be aggregated within a defined time window (e.g., last 5 minutes). If the producer's clock is significantly off, it can cause events that should fall into one time window to land in another, corrupting aggregation results.

To enforce event-time semantics and monitor clock drift effectively, we must first understand how clock skew impacts our system. Then, we will explore techniques for managing this skew through proper infrastructure design and monitoring practices.

#### Triage
What's Being Probed
Clock Skew Understanding: The interviewer is probing your understanding of how clocks in distributed systems can lead to issues with time-based operations.
Event-Time Semantics Enforcement: They are interested in your approach to ensuring that events are processed based on their actual occurrence times, not the timestamps reported by producers.
Drift Monitoring: The ability to detect and respond to clock skew is crucial for maintaining system correctness. The interviewer will be looking at how you would implement such monitoring.

#### First-Principles Solution
1. Understanding Clock Skew

In distributed systems, clocks can have varying accuracies due to network latency, hardware differences, or software implementation. For instance, a producer might report an event as occurring at 2023-11-15T14:30:00Z, but the actual time of production could be 14:29:55Z, leading to a 5-second skew.

2. Impact on Time-Windowed Aggregation

When events are aggregated within a window defined by timestamps, clock skew can cause confusion. For example:

A producer with a +30 second skew reports an event at 14:30:30Z but the actual production time was 14:29:50Z.
If the aggregation window is 5 minutes and we are aggregating events from 14:29:50Z to 14:35:00Z, the event will appear outside its intended window, corrupting our results.
3. Enforcing Event-Time Semantics

To enforce event-time semantics, a common approach is to use a watermark mechanism in the data flow pipeline. Watermarks help ensure that events are processed based on their actual occurrence time rather than the timestamp reported by producers. Here's how it works:

Watermark Calculation: Each producer generates watermarks indicating the latest known true time of its data.
Event Processing: Events are processed only if they occur before the current watermark in the consumer’s view.
4. Monitoring Drift

Monitoring clock drift is critical to ensure that the system remains correct over time. This involves:

Clock Synchronization: Using NTP (Network Time Protocol) or similar protocols to keep clocks synchronized.
Drift Detection: Implementing a mechanism to detect large deviations in producer clocks compared to a reference clock (like an external source of truth).
Alerts and Remediation: Setting up alerts for significant drift, and automating remediation steps such as restarting services or manually adjusting clocks.

#### Example
Workflow

Let's walk through an example workflow:

Producer Clock Skew: A producer reports an event at 2023-11-15T14:30:30Z with a +30-second skew.
Watermark Calculation: The producer calculates and sends the watermark as 2023-11-15T14:29:50Z.

#### Cons
umer Aggregation Window: Aggregates events from 2023-11-15T14:29:50Z to 2023-11-15T14:35:00Z.
Event Processing: The consumer processes the event if it is before the current watermark, avoiding the 5-second skew issue.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, clock skew can corrupt time-windowed aggregations by causing events to land in incorrect windows. To address this, enforcing event-time semantics with watermarks and monitoring drift through synchronization and alerting mechanisms is essential. Common mistakes include ignoring clock skew issues or using only system timestamps without proper validation.

#### Follow-ups
might involve discussing specific tools for watermark management (e.g., Apache Flink's event time processing), the impact of network latency on clock synchronization, or strategies for automated clock correction in a distributed environment.

**Comparison:**

```
Producer	Actual Time	Skew (s)	Reported Time
P1	2023-11-15T14:29:50Z	30	2023-11-15T14:30:30Z
P2	2023-11-15T14:31:20Z	-60	2023-11-15T14:25:20Z
Consumer Watermark	Reference Clock
2023-11-15T14:30:00Z	2023-11-15T14:30:00Z
```

**Code:**

```python
# Example Python code snippet for watermark calculation and event processing

import datetime
from abc import ABC, abstractmethod

class Watermark(ABC):
    @abstractmethod
    def get_watermark(self) -> datetime.datetime:
        pass

class ProducerWatermark(Watermark):
    def __init__(self, actual_time: datetime.datetime, skew_seconds: int):
        self.actual_time = actual_time
        self.skew_seconds = skew_seconds

    def get_watermark(self) -> datetime.datetime:
        return self.actual_time - datetime.timedelta(seconds=self.skew_seconds)

class ConsumerWatermark(Watermark):
    def __init__(self, reference_clock: datetime.datetime):
        self.reference_clock = reference_clock

    def get_watermark(self) -> datetime.datetime:
        return self.reference_clock
```

**Example input / output:**

```text
Input:
Producer1 reports an event at '2023-11-15T14:30:30Z' with a skew of +30 seconds.
Consumer has a reference clock at '2023-11-15T14:30:00Z'.

Output:
Watermark for Producer1 is calculated as '2023-11-15T14:29:50Z'.
Event is processed if it occurs before the consumer's watermark.

Dry-run: Event from producer at 2023-11-15T14:30:30Z (before 2023-11-15T14:29:50Z) -> NOT Processed.
```

**Data:**

```json
{
  "producer1": {
    "actual_time": "2023-11-15T14:29:50Z",
    "skew_seconds": 30,
    "reported_time": "2023-11-15T14:30:30Z"
  },
  "consumer_watermark": {
    "reference_clock": "2023-11-15T14:30:00Z"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Producer Reports Event
6. Calculate Watermark
7. Reported Time
8. Is Event Before Watermark?
9. Process Event
10. Do Not Process Event
11. Consumer Aggregates Events
12. Watermark Calculation
13. Reference Clock
14. Event Time < Watermark?
15. Process Event
16. Do Not Process Event

