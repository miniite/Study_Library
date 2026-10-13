---
type: "practice"
difficulty: "Expert"
---

# How would you rebuild a broken read model from the event log while the system keeps serving traffic?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Read Your Writes in Event Systems](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Read_Your_Writes_in_Event_Systems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To rebuild a broken read model from an event log without interrupting ongoing service, we need to carefully design a solution that minimizes downtime and ensures data consistency. At a high level, this involves capturing events in real-time or near-real-time, storing them for replay, and using a background job to reconstruct the read model. Here’s how I would approach this problem:

Understand the Problem: The core issue is to recover from a corrupted state of the read model without losing current service continuity. This involves ensuring that all new events are captured in real-time or stored for replay, and that a background job processes these events to update the read model.

#### Assumptions
and Constraints:

Events are logged in chronological order.
The system can handle occasional delays due to the background processing.
The event log is reliable and durable.

#### First-Principles Solution

Event Capture: Ensure that every read model update is logged as an event. This could be done using a distributed event store like Kafka, RabbitMQ, or AWS SNS/SQS.
Background Processing: Use a background job (e.g., a scheduled task or a message listener) to process the events from the log and apply them to the read model.
Read Model Updates: Implement an update mechanism that can apply events incrementally to the read model. This could be done using batch processing, streaming, or even a dedicated service.

#### Implementation
Steps:

Real-Time Event Logging: Ensure all system updates are logged as events in real-time. For example, if a write operation occurs, it should trigger an event log entry.
Event Storage and Processing: Use a reliable message queue to store these events. The backend service should listen for new events and process them incrementally.
Incremental Application of Events: Develop an algorithm that can apply events in order to the read model. This might involve reapplying all previous events or applying only the missing ones.

Recovery Process:

Offline Recovery: When a corruption is detected, trigger a background job to replay all unprocessed events from the log.
Online Recovery: If online recovery is needed, pause writes temporarily and process the backlog of events incrementally while ensuring minimal service disruption.

#### Closing Summary

The key takeaway is that by capturing and processing events in real-time or near-real-time, we can ensure that even if the read model gets corrupted, it can be reconstructed from a reliable log without interrupting ongoing service.

#### Common Mistakes
include underestimating the impact of event backlog on recovery time, not properly handling edge cases (like conflicting events), and failing to consider performance implications during the replay process.

**Comparison:**

```
Event Type	Description
UPDATE	Modifies an existing field in the read model.
CREATE	Adds a new entry to the read model.
DELETE	Removes an existing entry from the read model.
```

**Code:**

```python
# Example Python code for processing events in a background job
import json
from kafka import KafkaConsumer

def consume_events():
    consumer = KafkaConsumer('events', bootstrap_servers=['localhost:9092'])
    for message in consumer:
        event_data = json.loads(message.value)
        process_event(event_data)

def process_event(event):
    # Logic to apply the event to the read model
    pass
```

**Example input / output:**

```

Example Input/Output for Event Processing

PLAINTEXT
Copy
Input (Message from Kafka Consumer):
{
  "type": "UPDATE",
  "timestamp": "2023-10-01T14:30:00Z",
  "data": {
    "user_id": 1,
    "field": "name",
    "new_value": "John Doe"
  }
}

Output (Processed Event Applied to Read Model):
Read model updated for user_id 1 with new name 'John Doe'.
```

**Data:**

```json
{
  "event": {
    "type": "UPDATE",
    "timestamp": "2023-10-01T14:30:00Z",
    "data": {
      "user_id": 1,
      "field": "name",
      "new_value": "John Doe"
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Event Occurs
5. Is Event Logged?
6. Log Event
7. Background Job ProcessesEvents
8. Is Read Model Corrupted?
9. Read Model is ServedNormally
10. Trigger Background Job toReplay Events

