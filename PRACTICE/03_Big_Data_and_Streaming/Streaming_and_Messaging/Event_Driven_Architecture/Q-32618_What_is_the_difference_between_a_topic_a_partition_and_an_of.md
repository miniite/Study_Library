---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a topic, a partition, and an offset in Kafka in one clear sentence each?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Partition Keys](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Partition_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Kafka, topics are logical channels that allow messages to be published and subscribed to, akin to topics in a messaging system. Partitions, on the other hand, are the units of parallelism within a topic, enabling higher throughput and allowing for more concurrent readers or writers. Lastly, offsets represent the position in a partition, tracking the latest processed message so that consumers can continue from where they left off.

**Comparison:**

| Topic     | Partitions | Offsets |
|-----------|------------|---------|
| example-topic | [0, 1, 2]   | {0: 500, 1: 600, 2: 700} |

**Code:**

```python
# Example code to demonstrate the concept (not specific to Kafka but illustrative)
class MessageQueueConcept:
    def __init__(self):
        self.topics = {}  # Topics are key, partitions and offsets as values

    def add_topic(self, topic_name):
        self.topics[topic_name] = {"partitions": [], "offsets": {}}

    def add_partition(self, topic_name, partition_id):
        if topic_name in self.topics:
            self.topics[topic_name]["partitions"].append(partition_id)
            self.topics[topic_name]["offsets"][partition_id] = 0

# Example usage
mq_concept = MessageQueueConcept()
mq_concept.add_topic("example-topic")
mq_concept.add_partition("example-topic", 0)

print(mq_concept.topics)  # Output: {'example-topic': {'partitions': [0], 'offsets': {0: 0}}}
```

**Example input / output:**

```

Given the example topic and partitions:

PLAINTEXT
Copy
Input: Kafka topic = 'example-topic', Partitions = [0, 1, 2], Offsets for partition 0 = 500

Output:
- Topics: {'example-topic': {'partitions': [0, 1, 2], 'offsets': {0: 500, 1: 600, 2: 700}}}
```

**Data:**

```json
{
  "topics": {
    "example-topic": {
      "partitions": [0, 1, 2],
      "offsets": {0: 500, 1: 600, 2: 700}
    }
  }
}
```

**Flow:**

1. Topic
2. Partitions
3. Offsets

