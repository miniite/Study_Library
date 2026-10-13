---
type: "practice"
difficulty: "Deep"
---

# How would you consume a Kafka topic in a plain Python consumer (confluent-kafka or kafka-python) and commit offsets so a crash mid-batch doesn't skip or double-process?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Consumers in Python](<../../../../TOOLS/Python/Python_Language/Kafka_Consumers_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle consuming from a Kafka topic using Python, especially focusing on committing offsets to avoid data skew due to crashes, we'll leverage either confluent-kafka or kafka-python. The primary challenge is ensuring that we commit the offset only after successfully processing a message. This prevents messages from being missed (skipped) or processed multiple times (double-processing). Let's break down the solution step by step.

Firstly, the key idea here is to ensure idempotent processing of messages and correct handling of offsets in case of failures. We'll use Python's context management features and Kafka-specific functionalities for this task.

**Code:**

```python
from confluent_kafka import Consumer, KafkaError, TopicPartition

# Define the consumer configuration
conf = {
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'my_group',
    'auto.offset.reset': 'earliest'
}

consumer = Consumer(conf)

def process_message(msg):
    # Your message processing logic here
    print(f"Processing message: {msg.value().decode()}")
    return True

# Define the commit_offsets function to handle offset commits
def commit_offsets(consumer, topic_partitions):
    try:
        consumer.commitOffsets(topic_partitions)
        print("Offset committed successfully")
    except Exception as e:
        print(f"Failed to commit offsets: {e}")

# Define the main loop for consuming messages
while True:
    msg = consumer.poll(timeout=1.0)  # Poll for a message from Kafka

    if msg is None or msg.error():
        continue

    topic_partitions = [TopicPartition(msg.topic(), msg.partition(), msg.offset())]

    with consumer.acquireOffsetCommitLock(topic_partitions):
        try:
            result = process_message(msg)
            if result:  # If processing was successful
                commit_offsets(consumer, topic_partitions)
        except Exception as e:
            print(f"Error during message processing: {e}")
```

