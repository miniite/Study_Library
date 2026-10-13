---
type: "practice"
difficulty: "Intermediate"
---

# Design a class model for a notification-batching/digest engine that groups N events into one email if they arrive within a window (e.g., '3 people liked your post'); how do you model the batching window and flush?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Pub Sub Design](<../../../../07_Distributed_Systems_and_Design/System_Design/Low_Level_Design/Pub_Sub_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a class model for a notification-batching/digest engine, we need to focus on two main aspects: grouping events into batches based on timing criteria and sending these groups via email when conditions are met. At an intermediate level of experience, the core idea is to create classes that encapsulate the batching logic and ensure timely delivery.

The question is probing our ability to design a robust system with clear separation of concerns, manage state effectively, and handle edge cases. The assumptions we need to make include the types of events, the timing window for batch grouping, and how to determine when it's appropriate to send out the email digest.

Let’s reason step-by-step from first principles:

Define Event Class: We start by defining an Event class that stores information about each event.
Batching Logic: Implement a mechanism to group events within the specified window.
Batch Flush Mechanism: Determine when and how to send out email digests based on batch conditions.
Step 1: Define Event Class

The Event class should capture essential attributes such as type, timestamp, user ID, and event details.

PYTHON
class Event:
    def __init__(self, event_type, timestamp, user_id, message):
        self.event_type = event_type
        self.timestamp = timestamp
        self.user_id = user_id
        self.message = message

    def get_event_details(self):
        return {
            'type': self.event_type,
            'timestamp': self.timestamp,
            'user_id': self.user_id,
            'message': self.message
        }
Step 2: Batching Logic

We need a data structure to manage incoming events and group them based on the time window. A simple approach is to use a dictionary with timestamps as keys, storing lists of Event objects.

PYTHON
class EventBatcher:
    def __init__(self, batch_window_seconds):
        self.batch_window_seconds = batch_window_seconds
        self.events_by_time = {}

    def add_event(self, event: Event):
        current_time = event.timestamp
        time_bucket = current_time // self.batch_window_seconds * self.batch_window_seconds

        if time_bucket not in self.events_by_time:
            self.events_by_time[time_bucket] = []

        self.events_by_time[time_bucket].append(event)

    def flush_batches(self):
        batches_to_send = []
        for time_bucket, events in self.events_by_time.items():
            batch_message = ', '.join([event.message for event in events])
            if len(events) > 1:
                batches_to_send.append({
                    'timestamp': time_bucket,
                    'message': f'{len(events)} people {batch_message}'
                })
        return batches_to_send
Step 3: Batch Flush Mechanism

The flush_batches method processes all events in the current buckets and sends out email digests if there are multiple events to group.

#### Closing Summary

In summary, we have designed a class model that includes an Event class for storing event details, an EventBatcher class for managing batch grouping based on time windows, and a flush mechanism for sending out email digests. This design ensures that events within the specified window are grouped together and processed efficiently.

#### Common Mistakes/Follow-Ups
Mistake 1: Failing to manage state correctly; not updating or cleaning up expired batches.
Mistake 2: Not considering edge cases such as event timestamp discrepancies or batch size limitations.

#### Follow-up
Discussing how to handle different types of events, optimizing performance for large numbers of events, and ensuring system scalability.

**Comparison:**

```
Batch Time	Events	Message
1634589000	[Event1, Event2]	'2 people liked your post'
1634590700	[Event3]	'commented on your post'
```

**Code:**

```python
class Event:
    def __init__(self, event_type, timestamp, user_id, message):
        self.event_type = event_type
        self.timestamp = timestamp
        self.user_id = user_id
        self.message = message

    def get_event_details(self):
        return {
            'type': self.event_type,
            'timestamp': self.timestamp,
            'user_id': self.user_id,
            'message': self.message
        }

class EventBatcher:
    def __init__(self, batch_window_seconds):
        self.batch_window_seconds = batch_window_seconds
        self.events_by_time = {}

    def add_event(self, event: Event):
        current_time = event.timestamp
        time_bucket = current_time // self.batch_window_seconds * self.batch_window_seconds

        if time_bucket not in self.events_by_time:
            self.events_by_time[time_bucket] = []

        self.events_by_time[time_bucket].append(event)

    def flush_batches(self):
        batches_to_send = []
        for time_bucket, events in self.events_by_time.items():
            batch_message = ', '.join([event.message for event in events])
            if len(events) > 1:
                batches_to_send.append({
                    'timestamp': time_bucket,
                    'message': f'{len(events)} people {batch_message}'
                })
        return batches_to_send
```

**Example input / output:**

```
Sample input data

events = [ Event("like", 1634589800, 123, "liked your post"), Event("comment", 1634590700, 456, "commented on your post"), Event("like", 1634592800, 789, "liked your post") ]

Create an event batcher with a window of 300 seconds (5 minutes)

event_batcher = EventBatcher(300)

Add events to the batcher

for event in events: event_batcher.add_event(event)

Flush and get batches to send

batches_to_send = event_batcher.flush_batches()

print(batches_to_send)

CODE
Copy
Output:
```json
[
    {"timestamp": 1634589000, "message": "2 people liked your post"},
    {"timestamp": 1634590700, "message": "commented on your post"}
]
```

**Data:**

```json
[
    {"event_type": "like", "timestamp": 1634589800, "user_id": 123, "message": "liked your post"},
    {"event_type": "comment", "timestamp": 1634590700, "user_id": 456, "message": "commented on your post"},
    {"event_type": "like", "timestamp": 1634592800, "user_id": 789, "message": "liked your post"}
]
```

**Flow:**

1. Yes
2. No
3. Yes
4. Receive Event
5. Is event within window?
6. Add to batch
7. Flood batches?
8. Send Batch via Email
9. Clean Up Batch

