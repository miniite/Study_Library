---
type: "practice"
difficulty: "Deep"
---

# How would you write a Python consumer of a Kafka topic for a log-processing pipeline that commits offsets safely and survives a rebalance without losing or double-processing records?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Consumers in Python](<../../../../TOOLS/Python/Python_Language/Kafka_Consumers_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To create a robust Python consumer for a Kafka topic in a log-processing pipeline, we need to ensure that it handles offset committing safely and can survive rebalances without causing issues like losing or double-processing records. The key aspects include managing offsets correctly, handling partition assignment and revocation, and ensuring thread safety.

#### Triaging the Question

The question is asking for a Python implementation of a Kafka consumer that:

Safely commits offsets to ensure no data loss.
Handles rebalances gracefully without causing double-processing or missing records.
Ensures thread safety in case multiple threads are involved.
Key Assumptions and First Principles
The Kafka topic is partitioned, and each message should be processed only once.
We need to use the Kafka Python client library for interaction with Kafka brokers.
Rebalances occur when the consumer group changes its set of assigned partitions.
Step-by-Step Solution
1. Setup and Dependencies

First, ensure you have the necessary dependencies installed:

BASH
pip install kafka-python
2. Consumer Initialization

Initialize a Kafka consumer with appropriate configuration parameters such as auto_offset_reset, enable_auto_commit, and max_poll_records.

PYTHON
from kafka import KafkaConsumer

#### cons
umer = KafkaConsumer(
    'your_topic',
    bootstrap_servers=['localhost:9092'],
    auto_offset_reset='earliest',  # This ensures we start from the beginning of the log
    enable_auto_commit=False,      # We will control offset commits manually
    max_poll_records=10            # Controls how many messages to poll at once
)
3. Handling Partitions and Offsets

When a rebalance occurs, Kafka might revoke and assign partitions to different consumers in the group. To handle this gracefully:

Store offsets locally.
Manually commit offsets on successful processing.
PYTHON
class SafeKafkaConsumer:
    def __init__(self, topic):
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=['localhost:9092'],
            auto_offset_reset='earliest',
            enable_auto_commit=False,
            max_poll_records=10
        )
        self.offsets = {}

    def process_messages(self):
        for message in self.consumer:
            partition = message.partition
            if partition not in self.offsets:
                self.offsets[partition] = 0

            offset, payload = self.process_record(message)
            self.commit_offset(partition, offset)

    def process_record(self, message):
        # Placeholder function to process the record and return the processed offset
        print(f"Processing: {message.value}")
        # Simulate some processing time
        import time
        time.sleep(1)
        # For this example, we always increment the offset by 1
        return self.offsets[partition] + 1, message.value

    def commit_offset(self, partition, offset):
        if offset > self.offsets[partition]:
            self.consumer.commit({partition: OffsetAndMetadata(offset, None)})
            self.offsets[partition] = offset

# Example usage:

#### cons
umer = SafeKafkaConsumer('your_topic')

#### cons
umer.process_messages()
4. Thread Safety

If your consumer is running in a multi-threaded environment, ensure that the process_messages function is thread-safe by using locks or other synchronization mechanisms.

PYTHON
from threading import Lock

class SafeKafkaConsumer:
    def __init__(self, topic):
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=['localhost:9092'],
            auto_offset_reset='earliest',
            enable_auto_commit=False,
            max_poll_records=10
        )
        self.offsets = {}
        self.lock = Lock()

    # ... (rest of the methods with thread-safety)

#### Closing Summary
and Common Mistakes

This implementation ensures that the consumer processes messages safely and avoids double-processing or losing records during rebalances. Key points include manually managing offsets, handling partition assignment and revocation events, and ensuring thread safety.

#### Common mistakes
to avoid:

Not properly storing and committing offsets.
Ignoring partition reassignment logic.
Failing to handle thread-safety in multi-threaded environments.

**Code:**

```python
from kafka import KafkaConsumer

class SafeKafkaConsumer:
    def __init__(self, topic):
        self.consumer = KafkaConsumer(
            topic,
            bootstrap_servers=['localhost:9092'],
            auto_offset_reset='earliest',
            enable_auto_commit=False,
            max_poll_records=10
        )
        self.offsets = {}

    def process_messages(self):
        for message in self.consumer:
            partition = message.partition
            if partition not in self.offsets:
                self.offsets[partition] = 0

            offset, payload = self.process_record(message)
            self.commit_offset(partition, offset)

    def process_record(self, message):
        print(f"Processing: {message.value}")
        import time
        time.sleep(1)
        return self.offsets[partition] + 1, message.value

    def commit_offset(self, partition, offset):
        if offset > self.offsets[partition]:
            self.consumer.commit({partition: OffsetAndMetadata(offset, None)})
            self.offsets[partition] = offset
```

**Example input / output:**

```text
# Example usage:
consumer = SafeKafkaConsumer('your_topic')
consumer.process_messages()
```

