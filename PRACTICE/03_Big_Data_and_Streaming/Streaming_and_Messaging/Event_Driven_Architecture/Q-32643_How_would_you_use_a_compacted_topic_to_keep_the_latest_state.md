---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a compacted topic to keep the latest state per key, like a materialized cache of user profiles?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Partition Keys](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to use a compacted topic in Apache Kafka to maintain the latest state per key as a materialized cache of user profiles, we first need to understand what a compacted topic is and why it's suitable for this use case. A compacted topic ensures that messages are retained based on their offset or timestamp, rather than retaining all messages for a certain period. This makes it ideal for scenarios where you want to maintain the latest state of a key.

#### Triage

The question is asking how to leverage a compacted topic in Kafka to store user profiles and ensure that only the most recent profile per user is retained. The interviewer will be probing your understanding of Kafka’s message retention policies, specifically compacted topics, and your ability to design an event-driven architecture for state management.

First Principles
Understand Compacted Topics: A compacted topic in Kafka retains messages based on their offset or timestamp rather than their age. This means that only the latest message per partition key is retained.
Event-Driven Architecture (EDA): In an EDA, events are generated when state changes occur and are consumed by subscribers to update the current state.
State Management: The goal is to maintain a materialized cache of user profiles such that each user has only their latest profile stored.

#### Solution
Topic Configuration: Create a Kafka topic with compacted as its retention policy. This means that all messages for a given partition key will be retained, but duplicates (based on the key) are overwritten by later ones.
Producer Logic:
When a user profile is updated, generate an event containing the new state and publish it to the topic with the user’s ID as the key.

#### Cons
umer Logic:
Set up a consumer that subscribes to the topic. This consumer will process events in order, ensuring that only the latest message per key is processed.
Materialized Cache: Maintain a cache (e.g., Redis) of active user profiles. Whenever a new event is consumed and it has a newer state than what’s currently cached, update the cache.

#### Example
Workflow
User Profile Update:
A backend service detects that a user profile needs to be updated.
The service generates an event with the new profile data.
Kafka Event:
The event is published to the compacted topic with the user’s ID as the key.

#### Cons
umer Processing:
The consumer picks up the event and processes it, updating the cache if necessary.
Cache Update:
If the new profile data is more recent than what was in the cache, update the cache to reflect this state.

#### Closing Summary

Using a compacted topic in Kafka ensures that you maintain the latest state per user profile efficiently. This approach leverages Kafka’s event-driven capabilities and provides a scalable way to manage the materialized cache of user profiles without needing complex state tracking mechanisms.

#### Common Mistakes/Follow-Ups
Not Configuring the Topic Correctly: Ensure that the topic is configured with compacted retention policy.
Incorrect Keying: Make sure each message has a unique key (e.g., user ID) so that only one copy of it is retained per key.
Cache Synchronization Issues: Ensure proper synchronization between Kafka and the cache to avoid stale states.

**Comparison:**

```
Message Key	Event Data
12345	{"name": "Jane Doe", "email": "jane@example.com"}
```

**Code:**

```python
from kafka import KafkaProducer

# Example producer code
producer = KafkaProducer(bootstrap_servers=['localhost:9092'])
event_data = {"userId": "12345", "profile": {"name": "John Doe", "email": "john@example.com"}}
producer.send('user-profiles', value=event_data, key=b'12345')
```

**Example input / output:**

```text
- **Input**: `{"userId": "12345", "profile": {"name": "John Doe", "email": "john@example.com"}}` (keyed by `b'12345'`)
- **Output**: Latest profile data in the Kafka topic and cache.
```

**Data:**

```json
{
  "userId": "12345",
  "profile": {
    "name": "Jane Doe",
    "email": "jane@example.com"
  }
}
```

**Flow:**

1. User Profile Update
2. Generate Event
3. Kafka Producer
4. Kafka Topic (Compacted)
5. Consumer
6. Caching Layer (Redis)

