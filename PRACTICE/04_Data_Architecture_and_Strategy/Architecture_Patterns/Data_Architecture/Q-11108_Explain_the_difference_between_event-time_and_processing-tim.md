---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between event-time and processing-time and a time when confusing the two skewed your daily counts. How do you design around it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Streaming-first Data Architecture](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Streaming-first_Data_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Event-time and processing-time are fundamental concepts in data streaming, which can significantly affect how real-time metrics like daily counts are calculated. Event-time tracks the actual time when an event occurred, whereas processing-time is the timestamp associated with the moment an event is processed by a system. Misunderstanding these two can lead to skewed or inaccurate results, as illustrated in a common scenario involving daily user activity counts.

Let's dive into the details and explore a practical example where confusion between event-time and processing-time led to misleading daily counts.

**Comparison:**

```
Event Time	Processing Time
2023-04-01T15:30Z	2023-04-01T16:00Z
```

**Code:**

```python
# Example code for illustration purposes (actual code may vary)
from datetime import datetime

class Event:
    def __init__(self, timestamp):
        self.timestamp = datetime.fromisoformat(timestamp)

event1 = Event("2023-04-01 15:30")
event2 = Event("2023-04-01 16:00")

def process_events(events):
    daily_counts = {}
    for event in events:
        day_key = event.timestamp.date()
        if day_key not in daily_counts:
            daily_counts[day_key] = 0
        daily_counts[day_key] += 1
    return daily_counts

events = [event1, event2]
print(process_events(events))
```

**Example input / output:**

```text
# Example input and output for the process_events function
Input:
[
    Event("2023-04-01 15:30"),
    Event("2023-04-01 16:00")
]

Output:
{'2023-04-01': 1}
```

**Data:**

```json
[
  {"timestamp": "2023-04-01T15:30:00Z"},
  {"timestamp": "2023-04-01T16:00:00Z"}
]
```

**Flow:**

1. Event Time
2. Processing Time
3. Event Occurs
4. Is Event Time or ProcessingTime Used?
5. Count Incremented for CorrectDay
6. Count Incremented for CurrentDay

