---
type: "practice"
difficulty: "Deep"
---

# How would you choose the message key so a downstream warehouse merge/upsert stays correct even after you add partitions to the topic later?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Partitions and Ordering](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Partitions_and_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a downstream warehouse merge/upsert remains correct even after adding partitions to a Kafka topic, we need to carefully consider how message keys are chosen and how they interact with the partitioning strategy. The key idea is to design keys such that messages for the same logical entity always end up in the same partition, thereby maintaining consistency across merges or upserts.

#### Triaging the Question

The core of this question revolves around understanding Kafka's partitioning mechanism and ensuring that our message keys are robust enough to handle future changes. The interviewer is probing your knowledge of Kafka's internals, partitioning strategies, and how key design impacts downstream processing accuracy.

#### Assumptions

You have a Kafka topic with an initial set of partitions.
The downstream warehouse performs merges or upserts based on unique keys.
New partitions may be added to the Kafka topic in the future without changing the application logic significantly.

#### First-Principles Solution
1. Understand Partitioning Mechanism:

Kafka partitions messages into segments based on their key, ensuring that messages with the same key are always placed in the same partition. This helps in achieving distributed and fault-tolerant data processing.

2. Design Key to Ensure Consistency:

To maintain correct merges/upserts even after adding partitions, choose a key strategy that uniquely identifies each logical entity across all partitions. Here’s a step-by-step approach:

Unique Identifier Strategy: Use a unique identifier (UUID or ID) as the message key. This ensures that messages for the same entity are always grouped together.
Logical Key Strategy: Use a composite key based on business logic to ensure entities with the same attributes are consistently routed to the same partition.
3. Implement Consistent Hashing:

For distributed systems, consistent hashing can help in maintaining even distribution of keys across partitions. This reduces the impact of adding new partitions and minimizes data reassignment overhead.

#### Example
Composite Key Strategy

Let's assume we have an application that processes user events from a Kafka topic user_events. The downstream warehouse needs to merge or upsert these events based on unique users. A composite key could be:

PYTHON
class UserEventKey:
    def __init__(self, user_id):
        self.user_id = user_id

    def get_key(self):
        return f"user_{self.user_id}"

This ensures that all events for a given user_id are processed together, maintaining correctness in the downstream warehouse.

#### Common Mistakes
and Follow-Ups
Ignoring Partition Changes: Forgetting that new partitions might be added later can lead to incorrect processing.
Using Non-Unique Keys: Choosing keys that aren't unique across all entities can result in data loss or corruption.
Lack of Consistency: Not implementing consistent hashing strategies can cause issues during partition reassignment.

#### Follow-Ups

How would you handle scenarios where the number of partitions needs to be increased?
What are some best practices for designing message keys in Kafka?
How do you ensure that your key design scales well with growing data volume?

#### Closing Summary

Choosing appropriate message keys is crucial for maintaining consistency and correctness in downstream processing, especially when dealing with changes like adding new partitions. By using unique identifiers or composite keys based on business logic and implementing consistent hashing, we can ensure robust and scalable data processing.

**Comparison:**

```
Partition	Messages
0	user_1, user_2
1
2	user_1
```

**Code:**

```python
class UserEventKey:
    def __init__(self, user_id):
        self.user_id = user_id

    def get_key(self):
        return f"user_{self.user_id}"
```

**Example input / output:**

```text
Input (Kafka topic):
- Partition 0: [user_1, user_2]
- Partition 1: []

Output (Downstream Warehouse):
- User 1: login -> logout
- User 2: purchase

New partition added to Kafka Topic:
- Partition 2: [user_1]

New Output after adding new partition:
- User 1: login -> logout
- User 2: purchase
```

**Data:**

```json
{
  "user_events": [
    {"user_id": 1, "event_type": "login", "timestamp": "2023-10-01T08:00:00Z"},
    {"user_id": 2, "event_type": "purchase", "timestamp": "2023-10-01T09:00:00Z"},
    {"user_id": 1, "event_type": "logout", "timestamp": "2023-10-01T10:00:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. User Event Producer
4. Is Key Unique?
5. Kafka Topic with Partitioning
6. Downstream WarehouseMerge/Upsert Logic
7. Consistent Hashing Strategy
8. Kafka Topic with Partitioning
9. Downstream WarehouseMerge/Upsert Logic

