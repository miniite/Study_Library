---
type: "practice"
difficulty: "Intermediate"
---

# What happens if a consumer crashes after processing a message but before acknowledging it?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Offset Commit Modes](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Offset_Commit_Modes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an event-driven architecture using message queues, consumers process messages from a queue and typically acknowledge receipt of the message upon successful completion. If a consumer crashes without properly acknowledging that it has processed a message, the message is effectively "stuck" in the system because the acknowledgment mechanism fails to confirm the processing. This scenario can lead to duplicate processing if the message is requeued, or data loss if the message is not retried and lost.

#### Triage

The question asks what happens when a consumer crashes after receiving but before acknowledging a message. The interviewer is probing your understanding of:

Message queue semantics.
Acknowledgment mechanisms in event-driven architectures.
How to handle failures gracefully without causing data duplication or loss.

#### First-Principles Solution

To solve this, we need to understand the typical behavior of a message processing system and explore potential failure modes:

Message Reception: A consumer receives a message from a queue.
Processing Logic: The consumer processes the message.
Acknowledgment: The consumer acknowledges receipt once it has processed the message.

If the consumer crashes before acknowledging, the acknowledgment step does not occur, leading to a stale state where the message is neither considered processed nor available for reprocessing.

Scenario Analysis
Message Duplication: If messages are redelivered and processed again, it could lead to duplicate processing of the same data.
Data Loss: If no mechanism exists to retry or log unacknowledged messages, such messages may be lost entirely.

#### Solution

To mitigate these issues, several strategies can be employed:

Acknowledgment Mechanism: Use durable acknowledgments that ensure receipt is confirmed upon successful processing.
Dead Letter Queue (DLQ): Create a secondary queue where messages not acknowledged within a certain timeframe are moved for inspection or manual handling.
Idempotent Processing: Design the message processing logic to be idempotent, meaning that processing the same message multiple times has no effect beyond the first successful execution.

#### Example
Implementation

Suppose we have a system using RabbitMQ:

RabbitMQ Configuration:

YAML
exchanges:
  - name: messages
    type: direct
    durable: true

queues:
  - name: input_queue
    durable: true
    auto_delete: false
    arguments:
      x-dead-letter-exchange: dlx
      x-message-ttl: 60000 # 1 minute

bindings:
  - source: messages
    destination: input_queue
    routing_key: process

#### Cons
umer Implementation:

PYTHON
import pika

def consume_messages():
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()

    queue_name = 'input_queue'
    channel.queue_declare(queue=queue_name, durable=True)

    def callback(ch, method, properties, body):
        try:
            # Process the message
            process_message(body)
            ch.basic_ack(delivery_tag=method.delivery_tag)  # Acknowledge receipt
        except Exception as e:
            print(f"Error processing message: {e}")
            ch.basic_nack(delivery_tag=method.delivery_tag)  # Non-ack

    channel.basic_consume(queue=queue_name, on_message_callback=callback)
    channel.start_consuming()

def process_message(message):
    # Simulate some work
    print(f"Processing message: {message.decode()}")

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key takeaway is to ensure robust acknowledgment mechanisms and dead letter queues. Common mistakes include:

Ignoring non-delivery receipts.
Failing to implement retries or logging for unprocessed messages.

For follow-ups, the interviewer might ask about:

Retry Strategies: What retry policies would you apply?
Monitoring and Logging: How do you monitor and log failed message processing?

**Comparison:**

```
Strategy	Explanation
Acknowledgment	Ensures receipt confirmation upon successful processing.
Dead Letter Queue (DLQ)	Stores messages for inspection if not acknowledged within a timeframe.
```

**Code:**

```python
import pika

def consume_messages():
    connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
    channel = connection.channel()

    queue_name = 'input_queue'
    channel.queue_declare(queue=queue_name, durable=True)

    def callback(ch, method, properties, body):
        try:
            # Process the message
            process_message(body)
            ch.basic_ack(delivery_tag=method.delivery_tag)  # Acknowledge receipt
        except Exception as e:
            print(f"Error processing message: {e}")
            ch.basic_nack(delivery_tag=method.delivery_tag)  # Non-ack

    channel.basic_consume(queue=queue_name, on_message_callback=callback)
    channel.start_consuming()

def process_message(message):
    # Simulate some work
    print(f"Processing message: {message.decode()}")
```

**Example input / output:**

```text
Input: Message "Hello, world!"
Output:
Processing message: Hello, world!
```

**Data:**

```yaml
exchanges:
  - name: messages
    type: direct
    durable: true

queues:
  - name: input_queue
    durable: true
    auto_delete: false
    arguments:
      x-dead-letter-exchange: dlx
      x-message-ttl: 60000 # 1 minute

bindings:
  - source: messages
    destination: input_queue
    routing_key: process
```

**Flow:**

1. Yes
2. No
3. Message Received
4. Is Consumer Running?
5. Process Message
6. Acknowledge
7. Send to Dead Letter Queue(DLQ)

