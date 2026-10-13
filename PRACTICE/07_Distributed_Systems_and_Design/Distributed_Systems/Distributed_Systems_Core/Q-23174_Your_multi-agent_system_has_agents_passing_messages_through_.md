---
type: "practice"
difficulty: "Intermediate"
---

# Your multi-agent system has agents passing messages through a queue and one agent's malformed output poisons the consumer that parses it, stalling the pipeline. Explain the poison-message DLQ strategy and how you'd replay after fixing the producer.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Dead Letter Queues and Outbox](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Dead_Letter_Queues_and_Outbox.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of malformed messages in a multi-agent system where agents pass messages through a queue, we can implement a "poison message" Dead Letter Queue (DLQ) strategy. This involves creating a secondary queue that catches and isolates faulty or problematic messages for further inspection and handling. The key idea is to ensure the main processing pipeline remains unaffected by these issues, allowing us to debug and fix the producer without disrupting other agents in the system.

First, let's understand what the question is probing: it aims to assess your understanding of message queue mechanisms, error handling strategies, and how you would implement robustness in a distributed system. We will assume several key points:

The system uses a typical publish-subscribe model with RabbitMQ or Kafka.
Messages are produced by agents and consumed by other agents or services.
A malformed message causes the consumer to stall, affecting the entire pipeline.

The strategy involves setting up two main queues: one for regular messages and another for "poison" messages. Here’s how we would implement this:

Define Queues: Create a primary queue (e.g., mainQueue) where agents publish their messages.

#### Cons
umer Setup: Configure the consumer to listen on both mainQueue and a poison message queue (poisonQueue). If it encounters an error while processing a message, instead of failing the entire pipeline, it should send the problematic message to poisonQueue.
Error Handling Mechanism: Implement a mechanism within the producer that handles malformed messages by wrapping them in specific headers or tags. The consumer can use these headers to detect and route such messages to poisonQueue.

Let’s dive into the first-principles solution step-by-step.

Step 1: Define the Queues

We set up two queues:

mainQueue: For normal message processing.
poisonQueue: For isolating problematic or malformed messages.
Poison Queue
poisonQueue
Main Queue
mainQueue
Step 2: Consumer Implementation

The consumer should be configured to listen on both mainQueue and poisonQueue. When it receives a message, it checks for errors. If an error occurs (e.g., due to malformed data), the message is sent to poisonQueue.

PYTHON
# Pseudocode for consumer
while True:
    try:
        message = get_message_from_queue(mainQueue)
        process_message(message)
    except Exception as e:
        send_to_poison_queue(poisonQueue, message, str(e))
Step 3: Producer Error Handling

The producer should be aware of the potential issues and handle them appropriately. It can use specific headers or tags to wrap problematic messages before sending them.

PYTHON
# Pseudocode for producer
def send_message(message):
    if is_malformed(message):
        tag = "poison"
        message_with_tag = {message, "tag": tag}
        send_to_queue(mainQueue, message_with_tag)
    else:
        send_to_queue(mainQueue, message)
Step 4: Replay Mechanism

Once the producer issue is fixed and the poisonQueue has been reviewed, we can replay messages from it back to mainQueue. This ensures that no important data is lost due to temporary issues.

PYTHON
# Pseudocode for replay mechanism
while poisonQueue.has_messages():
    message = get_message_from_queue(poisonQueue)
    process_message(message)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the DLQ strategy helps in isolating problematic messages without disrupting the main processing pipeline. By setting up a separate queue for these "poison" messages, we ensure robustness and maintain system reliability.

#### Common mistakes
to avoid include:

Not having a clear error handling mechanism within producers.
Failing to set up dedicated queues for poison messages.
Overlooking the need for replay mechanisms after fixing issues.

For follow-ups, you might be asked about scaling concerns, security considerations, or integration with existing systems. Make sure to address these points as needed during the interview.

**Code:**

```python
# Pseudocode for consumer and producer
```

