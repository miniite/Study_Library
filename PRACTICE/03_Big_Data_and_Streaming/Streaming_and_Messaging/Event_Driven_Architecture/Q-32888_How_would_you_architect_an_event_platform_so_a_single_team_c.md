---
type: "practice"
difficulty: "Deep"
---

# How would you architect an event platform so a single team can safely run a 'shadow' consumer against production event streams to prototype a feature, with zero risk of side effects, resource starvation, or offset interference on the real consumers?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Breaking Schema Changes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Fat_vs_Thin_Events/Breaking_Schema_Changes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To architect an event platform that allows a single team to safely run a 'shadow' consumer against production event streams without causing any side effects, resource starvation, or offset interference on the real consumers, we need to consider several critical aspects of event-driven architectures (EDA) and message queues. The key is to ensure isolation, fault tolerance, and seamless integration. Here’s how we can approach this:

Isolation: The shadow consumer must not interact with any part of the production system that could cause unintended side effects.
Fault Tolerance: Ensure the platform can handle failures gracefully without impacting the primary consumers or causing data loss.
Resource Management: Prevent resource starvation by controlling the rate and volume of events processed by the shadow consumer.

The core idea is to design a robust, isolated environment for the shadow consumer using techniques such as topic partitioning, dead-letter queues (DLQs), and rate limiting.

**Comparison:**

```
Topic Partition	Consumers
0	Shadow Consumer, Primary Consumers
1	Shadow Consumer, Primary Consumers
...	...
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Event Producer] --> B{Topic Partition};
    B --> C1[Shadow Consumer];
    B --> C2[Primary Consumers];
    C1 --> D[Dead-Letter Queue];
```

**Example input / output:**

```

Input Event Payload (JSON):

JSON
Copy
{
  "event_id": "e123",
  "timestamp": "2023-10-01T12:00:00Z",
  "data": {
    "user_id": "u456",
    "action": "purchase"
  }
}

Output to Primary Consumers (JSON):

JSON
Copy
{
  "event_id": "e123",
  "processed_timestamp": "2023-10-01T12:00:05Z"
}

Output to Shadow Consumer (JSON):

JSON
Copy
{
  "event_id": "e123",
  "shadow_processed_timestamp": "2023-10-01T12:00:08Z"
}
```

**Data:**

```yaml
topic_partitions:
  - id: 0
    consumers: [C1, C2]
dead_letter_queue: DLQ-0
```

**Flow:**

1. Event Producer
2. TopicPartition-0
3. PrimaryConsumer
4. ShadowConsumer
5. DeadLetterQueue

