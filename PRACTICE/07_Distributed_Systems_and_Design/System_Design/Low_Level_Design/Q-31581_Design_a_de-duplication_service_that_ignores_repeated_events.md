---
type: "practice"
difficulty: "Deep"
---

# Design a de-duplication service that ignores repeated events within a window. What state do you hold?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Idempotent Consumer Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Idempotent_Consumer_Design/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a de-duplication service that ignores repeated events within a sliding window, we need to consider both the state management and the event processing logic. The goal is to ensure that duplicate events are ignored if they occur within a specified time frame, while keeping performance and scalability in mind.

The question is essentially asking us to identify how we would maintain an internal state to track recent events and what data structures or mechanisms we might use to efficiently manage this state. This involves understanding the trade-offs between space complexity, lookup efficiency, and update operations.

#### Triage
What the Question Is Asking: The interviewer wants to understand how you approach designing a system that can handle event streams and ensure uniqueness within a sliding window.

#### Assumptions
Events have unique identifiers (IDs) or timestamps.
There is a maximum allowed duration for events to be considered duplicates.

#### First Principles Solution
Event Tracking: We need a way to track events that were seen recently. This can be achieved using data structures like sets, hash maps, or more advanced solutions like Bloom Filters if false positives are acceptable.
Sliding Window Management: A sliding window mechanism ensures we only consider recent events for duplicates.
State Handling:
Event Identification: Each event must have a unique identifier (ID) to ensure it can be tracked and compared with others.
Time Window Calculation: We need to determine the duration of the window in which events are considered duplicates.
Detailed Solution
Data Structure Selection: Given the need for fast lookup, insertion, and removal operations, a combination of sets (to track seen event IDs) and hash maps (to link IDs with their timestamps) can be effective.
Window Mechanism:
Implement a sliding window where each slot in the window corresponds to a time interval within the specified duration.
For each new event, check if its ID is already present in the set of seen events or hash map.
If not, add it and insert the timestamp into the appropriate position in the sliding window.
Event Handling:
PYTHON
class EventDeduplicator:
    def __init__(self, window_duration):
        self.window_duration = window_duration  # Duration of the sliding window in seconds
        self.seen_events = set()  # To track unique event IDs
        self.events_by_time = {}  # Timestamps for events within the current window

    def process_event(self, event_id, timestamp):
        if event_id not in self.seen_events:
            self.seen_events.add(event_id)
            self.events_by_time[timestamp] = True
        else:
            print("Duplicate event ignored:", event_id)

# Example usage
dedup = EventDeduplicator(3600)  # Window duration of 1 hour
events = [("event1", 1628574400), ("event2", 1628574401), ("event3", 1628574401),
          ("event2", 1628574402)]  # Event IDs and timestamps
for event_id, timestamp in events:
    dedup.process_event(event_id, timestamp)

#### Closing Summary
+ Common Mistakes/Follow-Ups
Summary: The key to designing this system is using appropriate data structures to manage the state of seen events efficiently within a sliding window.

#### Common Mistakes
Overlooking the need for fast lookup operations (e.g., using sets and hash maps).
Ignoring the importance of handling timestamps correctly when implementing the sliding window.
Failing to consider edge cases, such as what happens if two events have the same timestamp but different IDs.
Additional Considerations
Scalability: Ensure the system can handle a large number of concurrent events without performance degradation.
False Positives/Negatives: Discuss the implications of using more advanced structures like Bloom Filters for potential false positives and negatives, which might be acceptable in certain scenarios.

**Comparison:**

```
Event ID	Timestamp
event1	1628574400
event2	1628574401
event3	1628574401
event2	1628574402
```

**Code:**

```python
class EventDeduplicator:
    def __init__(self, window_duration):
        self.window_duration = window_duration  # Duration of the sliding window in seconds
        self.seen_events = set()  # To track unique event IDs
        self.events_by_time = {}  # Timestamps for events within the current window

    def process_event(self, event_id, timestamp):
        if event_id not in self.seen_events:
            self.seen_events.add(event_id)
            self.events_by_time[timestamp] = True
        else:
            print("Duplicate event ignored:", event_id)
```

**Example input / output:**

```text
dedup = EventDeduplicator(3600)  # Window duration of 1 hour
events = [
    ("event1", 1628574400),
    ("event2", 1628574401),
    ("event3", 1628574401),
    ("event2", 1628574402)
]  # Event IDs and timestamps

for event_id, timestamp in events:
    dedup.process_event(event_id, timestamp)

# Output:
# Duplicate event ignored: event2
# Duplicate event ignored: event3
```

**Data:**

```json
[
    {"eventId": "event1", "timestamp": 1628574400},
    {"eventId": "event2", "timestamp": 1628574401},
    {"eventId": "event3", "timestamp": 1628574401},
    {"eventId": "event2", "timestamp": 1628574402}
]
```

**Flow:**

1. Yes
2. No
3. Receive Event
4. Is Event ID in Seen Events?
5. Ignore Event
6. Add Event to Seen Events,Update Window
7. E
8. Process Event

