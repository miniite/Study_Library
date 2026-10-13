---
type: "practice"
difficulty: "Deep"
---

# A user updates their profile but the search index shows old data for a few seconds. How do you explain and handle this?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Lag-based Consumer Autoscaling](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Lag-based_Consumer_Autoscaling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an event-driven architecture, when a user updates their profile, we need to ensure that all downstream systems, such as the search index, are updated in real-time or near-real-time. However, there can be latency issues where the search index might show old data briefly for a few seconds after the profile update. This question requires understanding how message queues work and how they can mitigate these kinds of inconsistencies.

The core idea here is to use an asynchronous event-driven system with durable message queues to decouple the user update process from the indexing process, ensuring eventual consistency without making clients wait.

#### Triage
Understanding the Problem
Question Probing: The interviewer wants to understand your knowledge of how event-driven systems and message queues can handle data consistency issues.

#### Assumptions
User updates are written to a database.
A search index needs to be updated after these changes.
There is some delay in updating the search index due to various factors such as network latency, processing time, or backpressure.

#### First Principles Solution
User Profile Update:
When a user updates their profile, this event is first written to a database (e.g., PostgreSQL).
Message Queue:
The database change triggers an event that gets pushed onto a message queue (e.g., RabbitMQ or Kafka).
Indexing Service:
A background service listens to the message queue and updates the search index accordingly.
Latency Management:
The indexing process should be designed with retries, backpressure handling, and rate limiting to manage potential delays.
Detailed Steps
Database Write:
User profile update occurs and is written directly into a database. This ensures that the data is immediately available for other systems to read (primary key).
Message Queue Trigger:
A database trigger or application-level event listener sends an event to a message queue, indicating that a user profile has been updated.
Message Processing:
The indexing service consumes messages from the queue and updates the search index.
Handling Delays:
If there is a delay in processing (e.g., due to high load), the message will be reprocessed once the system recovers.
Edge Cases and Failures
Database Failure: Ensure that the database write is durable, using transactions or journaling mechanisms.
Queue Backpressure: Design the indexing service to handle large volumes of messages with appropriate backpressure handling (e.g., rate limiting).
Index Update Failure: Implement retry logic for index updates to ensure eventual consistency.

#### Closing Summary

By implementing a message queue-based architecture, we can decouple the user update process from the search index update process. This ensures that users see their updated profile immediately while also preventing inconsistent data displays in searches. The key components include reliable database writes, durable messages, and resilient indexing services with proper error handling.

#### Common Mistakes/Follow-ups
Not Using Message Queues: Simply relying on synchronous calls between systems can lead to cascading failures.
Ignoring Backpressure: Not managing backpressure effectively can lead to system-wide delays.
Insufficient Retry Mechanisms: Failure to implement retries for critical updates can result in data inconsistencies.

**Comparison:**

| Time | Event            | System           |
|------|------------------|-----------------|
| 0    | Profile update   | Database         |
| 1ms  | Message published| Queue           |
| 5s   | Index updated    | Search Service   |

**Code:**

```python
# Example of a simple database write and message queue interaction

import psycopg2
from rabbitmq import RabbitMQPublisher  # Hypothetical RabbitMQ client library

def update_user_profile(user_id, new_data):
    try:
        # Step 1: Write to database
        with psycopg2.connect(dbname="user_db") as conn:
            with conn.cursor() as cur:
                query = "UPDATE users SET data=%s WHERE id=%s"
                cur.execute(query, (new_data, user_id))

        # Step 2: Publish message to queue
        publisher = RabbitMQPublisher()
        payload = {"action": "update_user_profile", "user_id": user_id}
        publisher.publish(payload)
    except Exception as e:
        print(f"Error: {e}")
```

**Example input / output:**

```text
Input:
- User updates profile with new name.

Output (dry-run):
1. Database write to users table.
2. Message queue event published with user ID and action.
3. Indexing service consumes message and updates search index.
4. User sees updated profile immediately, while the search index is still old for a few seconds.
```

**Data:**

```json
{
  "database_update": {
    "table": "users",
    "fields": ["id", "data"],
    "values": [{"user_id": 1, "new_data": {"name": "John Doe"}}]
  },
  "message_queue_event": {
    "action": "update_user_profile",
    "user_id": 1
  }
}
```

**Flow:**

1. User Updates Profile
2. Database Write
3. Message Queued
4. Index Update Service
5. Search Index Updated
6. User Sees Updated Profile
7. Old Data Visible for Seconds

