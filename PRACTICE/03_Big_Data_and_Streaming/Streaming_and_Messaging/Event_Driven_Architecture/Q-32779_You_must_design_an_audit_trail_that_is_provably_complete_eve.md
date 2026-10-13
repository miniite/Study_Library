---
type: "practice"
difficulty: "Deep"
---

# You must design an audit trail that is provably complete: every state change is captured as an event with no gaps. How do you detect and alert on a missing event in the sequence?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event-driven Order Processing](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Event-driven_Order_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure an audit trail is provably complete, we need to design a system that captures every state change as an event and can detect any gaps or missing events. This involves understanding the nature of message queues and event-driven architectures (EDA). Here’s a step-by-step approach:

#### Overview
We will use a combination of Kafka for messaging, a database for storing audit logs, and a monitoring system to alert on missing events.
Understanding the Problem: The goal is to detect gaps in the sequence of state changes. This requires tracking event IDs and ensuring they are contiguous without any missing numbers.
First Principles Analysis:
Message Queue Design: We use Kafka’s topic partitions to ensure each event gets a unique ID.
Audit Logging: We store audit logs in a relational database for easy querying and analysis.
Monitoring System: A separate system monitors the sequence of events and alerts if any are missing.

#### Triaging the Question

What the Interviewer is Probing:

Your understanding of EDA principles.
How to ensure data integrity and consistency in a distributed environment.
The ability to design fault-tolerant systems with robust error detection mechanisms.

#### Assumptions

We are using Kafka for reliable messaging.
The state changes are critical enough that gaps need to be detected and acted upon promptly.

#### Solution

Kafka Topic Design:

Each event is assigned a unique sequence ID, which is part of the message payload.
Use Kafka's partitioning mechanism to ensure each event gets a unique position within its topic.

Audit Log Storage:

Store audit logs in a database with an event_id column.
Ensure that the sequence IDs are strictly increasing and continuous.

Monitoring System:

A separate service or microservice monitors the Kafka topic for new events.
It checks the sequence of event IDs to ensure no gaps.
If a gap is detected, it triggers an alert via a notification system (e.g., Slack, email).
Detailed Solution

Kafka Topic Setup:

Produce
Topic 1
Partition 0
Partition 1
Event ID: 1
Event ID: 2

Database Schema:

AuditLog
+id
+event_id
+timestamp
+state_change_details
event_id is a primary key to ensure uniqueness.
state_change_details captures the specifics of the state change.

Monitoring Service:

The monitoring service subscribes to the Kafka topic and processes incoming events.
Gap detected
No gap
Monitor
Listen for new messages
Check sequence ID
Alert & Log Error
Continue Monitoring

Alert Mechanism:

Use a notification system to alert the appropriate team or person responsible for investigating the missing event.
Monitoring Service
Notification System(Slack/Email)
Notify: Missing Event

#### Closing Summary

By using Kafka for reliable messaging, a database to store audit logs, and a monitoring system to check event sequences, we can ensure that every state change is captured and any gaps are detected. This design leverages the strengths of EDA while providing robust error detection mechanisms.

#### Common Mistakes/Follow-Ups

Forgetting to use unique sequence IDs for events.
Not implementing proper database constraints on event_id.
Neglecting to have a robust alert mechanism in place.

**Comparison:**

| AuditLog Table |
| --- |
| id (int)        |
| event_id (int)  |
| timestamp      |
| state_change_details (text) |

**Code:**

```python
# Example Kafka Consumer Code
from kafka import KafkaConsumer

consumer = KafkaConsumer('audit-topic', bootstrap_servers=['localhost:9092'])
for message in consumer:
    event_id = message.value['event_id']
    # Process and store the event
```

**Example input / output:**

```text
Input:
Event with event_id = 102

Output:
Audit log entry stored in database, and no alert is triggered.

Input:
Missing event with expected event_id = 103

Output:
Alert sent via Slack or email to the support team.
```

**Data:**

```json
{
  "audit_log_table": {
    "id": int,
    "event_id": int,
    "timestamp": datetime,
    "state_change_details": string
  }
}
```

**Flow:**

1. Gap Detected
2. No Gap
3. Produce Event
4. Assign Unique ID
5. Kafka Topic - Partition 0
6. Event ID: 1234567890
7. Database Log Entry
8. Monitoring Service
9. Check Sequence
10. Alert System
11. Continue Monitoring

