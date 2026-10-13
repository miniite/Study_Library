---
type: "practice"
difficulty: "Deep"
---

# How do you commit offsets safely in a Python Kafka consumer so a crash mid-batch doesn't skip or duplicate messages, and what's the trade-off of auto vs manual commit?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Consumers in Python](<../../../../TOOLS/Python/Python_Language/Kafka_Consumers_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure safe offset committing in a Python Kafka consumer, we need to avoid both message skipping and duplication during crashes mid-batch. This involves understanding how Kafka offsets work and choosing between automatic or manual commits with appropriate safeguards.

Kafka consumers can commit offsets automatically after each batch of messages (using enable.auto.commit), but this approach is risky as it may lead to duplicated messages if the consumer crashes before committing an offset. Alternatively, we can implement manual commits using commitSync() or commitAsync(). This requires more careful handling to ensure that the consumer does not skip messages upon recovery.

Let's break down the steps to achieve safe offset committing and discuss the trade-offs between auto vs manual commits.

#### Triaging the Question
Understand the Core Problem: Ensure the consumer does not skip or duplicate messages upon a crash.
Identify Commit Mechanisms:
Auto-commit: Simple but risky, especially in fault-tolerant environments.
Manual commit: More control but requires more robust handling.

#### Assumptions
and Constraints
The Kafka cluster is reliable and the consumer can communicate with it.
We are using Python's confluent-kafka library for interactions with Kafka.
Error handling and recovery mechanisms are in place to manage crashes gracefully.

#### First Principles Solution
Auto Commit (Risky but Simple)

Auto-commit involves setting enable.auto.commit = True on the consumer configuration. This means the consumer will commit offsets periodically or immediately after consuming messages, which simplifies offset management. However, this can lead to duplicates if a crash occurs before the next auto-commit.

PYTHON
from confluent_kafka import Consumer

# Configure the consumer with auto-commit enabled

#### cons
umer_config = {
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'my_group',
    'enable.auto.commit': True  # This will automatically commit offsets
}

#### cons
umer = Consumer(consumer_config)
Manual Commit (Safer but Complex)

Manual commits require more sophisticated handling to ensure that messages are not skipped or duplicated. We can use commitSync() and commitAsync() methods, which allow us to explicitly control when and how offsets are committed.

PYTHON
from confluent_kafka import Consumer, KafkaException, KafkaError

# Configure the consumer without auto-commit

#### cons
umer_config = {
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'my_group',
    'enable.auto.commit': False  # This will disable automatic commits
}

#### cons
umer = Consumer(consumer_config)

try:
    while True:
        msg = consumer.poll(timeout=1.0)  # Poll for messages with a timeout

        if msg is None:
            continue

        if msg.error():
            if msg.error().code() == KafkaError._PARTITION_EOF:
                continue
            else:
                raise KafkaException(msg.error())

        # Process the message here...

        # Commit offsets manually after processing each batch

#### cons
umer.commitSync()

#### Trade-Offs
Between Auto vs Manual Commits

Simplicity:

Auto-commit: Easier to implement and manage.
Manual commit: More complex but offers better control.

Risk of Duplication:

Auto-commit: Higher risk due to lack of explicit offset management.
Manual commit: Lower risk as offsets are committed only after processing is complete, reducing duplication chances.

Error Handling and Recovery:

Auto-commit: Less robust error handling required; recovery from crashes may be complex.
Manual commit: Requires proper error logging and retry mechanisms to handle crashes gracefully.

Latency and Throughput:

Auto-commit: Can reduce latency as offsets are committed more frequently.
Manual commit: May increase latency but can be optimized with advanced techniques like async commits.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, while auto-commit simplifies implementation, it introduces a higher risk of message duplication. Manual commits provide safer handling by ensuring offsets are only committed after processing is complete, but require more complex error management and recovery logic.

#### Common mistakes
include:

Failing to handle errors properly during manual commits.
Not implementing robust retry mechanisms for failed commits.
Overlooking the need for proper offset logging and tracking.

#### Follow-up
questions might include how to implement async commits, handling leader changes in Kafka clusters, or optimizing commit strategies under varying workload conditions.

**Code:**

```python
from confluent_kafka import Consumer

# Example of auto-commit configuration
consumer_config = {
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'my_group',
    'enable.auto.commit': True  # This will automatically commit offsets
}
```

