---
type: "practice"
difficulty: "Intermediate"
---

# You need a Python consumer that ingests metrics from a Kafka topic into a detector without falling behind. How do you commit offsets safely so a crash mid-batch doesn't drop or reprocess telemetry?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Kafka Consumers in Python](<../../../../TOOLS/Python/Python_Language/Kafka_Consumers_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of safely committing offsets in a Python consumer that ingests metrics from a Kafka topic, we need to ensure that the consumer processes messages reliably and does not miss any data upon recovery or restart. This involves understanding how to manage offset commits and handle batch processing efficiently.

The key challenge here is to design a mechanism where, if the consumer crashes mid-batch, it can resume processing from the correct point without reprocessing already ingested data or missing new incoming telemetry. We'll reason through this step-by-step using first principles and implement a robust solution in Python.

First, let's break down what we need:

Message Processing: The consumer must process messages in batches.
Offset Committing: Offsets should be committed only after all messages in the current batch are successfully processed.
Resilience: In case of a crash, the consumer should be able to recover and continue from where it left off.

Here’s how we can achieve this:

Triaging the Problem

The question is probing our understanding of Kafka offset management and our ability to implement robust error handling in Python. We need to ensure that:

Offsets are committed only after successful message processing.
The consumer resumes processing correctly after a crash, without reprocessing or missing any data.

#### Assumptions
Messages are ingested from a single partition of the Kafka topic.
We use the confluent-kafka library for interacting with Kafka in Python.
Each message contains a unique identifier and possibly additional metadata.

#### First-Principles Solution
Step 1: Define Message Processing Logic

We need to define a function that processes each message within a batch. This could involve complex logic depending on the nature of the metrics (e.g., aggregating data, applying transformations).

PYTHON
def process_message(message):
    # Example processing logic
    metric = parse_metric(message.value)  # Parse and validate the message
    detector.detect(metric)  # Process the metric with the detector
Step 2: Implement Batch Processing and Offset Management

We’ll use a loop to consume messages in batches, process them, and commit offsets only if all messages are successfully handled.

PYTHON
from confluent_kafka import Consumer

# Kafka consumer setup

#### cons
umer = Consumer({
    'bootstrap.servers': 'localhost:9092',
    'group.id': 'my-group',
    'auto.offset.reset': 'earliest'
})

# Subscribe to the topic
topic_name = 'metrics-topic'

#### cons
umer.subscribe([topic_name])

offsets_to_commit = []

try:
    while True:
        msg = consumer.poll(timeout=1.0)
        if msg is None:
            # No message available for consumption, continue processing other batches
            continue

        if msg.error():
            print(f"Error: {msg.error()}")
            # Handle errors appropriately, e.g., retry or log the error

        offset = (topic_name, msg.offset())
        offsets_to_commit.append(offset)

        process_message(msg)
except KeyboardInterrupt:
    pass

finally:
    # Commit offsets only after all messages in this batch are processed

#### cons
umer.commit(asynchronous=False, offsets=offsets_to_commit)
Step 3: Handling Crashes and Recovery

In case of a crash during message processing, the consumer will not commit any offset until it processes all messages. Upon recovery, it will resume from the last committed offset.

#### Closing Summary
+ Common Mistakes/Follow-Ups

To ensure reliability:

Offset Management: Only commit offsets after successful processing.
Error Handling: Handle errors gracefully to prevent partial batches.
Logging and Monitoring: Implement logging and monitoring to track consumer status and error conditions.

#### Common mistakes
include:

Committing offsets too early, leading to data loss or reprocessing.
Not handling errors properly, causing the entire batch to fail.
Ignoring asynchronous offset commits which can lead to race conditions during recovery.
Further Considerations

For a more robust solution, consider implementing retries for processing failures and managing larger batches more efficiently. Additionally, you might want to explore strategies like idempotent message processing or using transactional semantics if your use case requires it.

