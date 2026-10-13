---
type: "practice"
difficulty: "Deep"
---

# You need to guarantee ordering across a create, N updates, and a delete for the same entity, but the create and delete are produced by different services. How do you keep them ordered end to end?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event-driven Order Processing](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Event-driven_Order_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure ordering of operations (create, N updates, and delete) for the same entity across two different services—where creates and deletes are handled by separate services—we need a robust event-driven architecture that guarantees message order through an intermediary message queue system.

First, let's clarify what we're aiming to achieve:

Guaranteed Order: The operations (create, updates, delete) should be processed in the exact sequence they were produced.
Service Independence: The create and delete services operate independently but need to coordinate their actions for a single entity.
Message Queue System: We use an intermediary message queue that ensures messages are processed strictly in order.

Now, let’s break down the solution step by step:

Identify Messages:

Create Message (CM): Produced when a new entity is created.
Update Messages (UM): Produced for each update operation on an existing entity.
Delete Message (DM): Produced when a delete operation is initiated.

Message Queue Design:

Single Queue: Use a single queue to store all messages related to the entity in order of their creation.
Ordering Mechanism: Ensure that no other message can be inserted between CM and DM or any pair of consecutive UM's.

Publish-Subscribe Model:

The create service publishes the CM along with all subsequent UM’s into a queue.
The delete service only needs to publish its DM without interfering with the order of messages already in the queue.

#### Cons
umer Logic:

A single consumer or multiple consumers (depending on scalability requirements) will process these messages from the queue.
Ensure that no two consumers can read from this queue concurrently to avoid any race conditions and maintain strict ordering.

Handling Deletes:

The delete service should not modify the message order in the queue but only signal its intent by publishing a DM.
The consumer(s) will then handle both CM and DM as part of their processing logic, ensuring that the delete action is applied after all updates have been processed.

Error Handling:

Implement retry mechanisms for transient failures without affecting order.
Handle duplicate messages to ensure they are processed only once in sequence.

Scalability Considerations:

Use scalable message queues like AWS SQS, RabbitMQ, or Kafka that support ordering and can handle high throughput.
Partition the queue if necessary to distribute load among multiple consumers without breaking order guarantees.

By following these steps, we ensure that all operations for a single entity are processed in the correct sequence regardless of which service initiates them. This design leverages the strengths of message queues to provide a scalable and reliable solution.

**Comparison:**

| Message Type | Sequence |
|--------------|----------|
| Create       | 1        |
| Update       | >1       |
| Delete       | n+1      |

**Code:**

```python
# Example Python pseudocode for processing messages from queue

import boto3  # AWS SQS client

def process_message_queue(queue_url):
    sqs = boto3.client('sqs')

    while True:
        response = sqs.receive_message(
            QueueUrl=queue_url,
            AttributeNames=['All'],
            MessageAttributeNames=['All'],
            MaxNumberOfMessages=10,
            WaitTimeSeconds=20
        )

        if 'Messages' not in response:
            continue

        for message in response['Messages']:
            # Process each message here, ensuring strict order
            process_message(message)

            # Delete the processed message from queue
            sqs.delete_message(
                QueueUrl=queue_url,
                ReceiptHandle=message['ReceiptHandle']
            )
```

**Example input / output:**

```text
Input:
- Create message (CM)
- Update messages (UM1, UM2, ...)
- Delete message (DM)

Output:
- CM -> Processed by consumer
- UM1 -> Processed after CM
- UM2 -> Processed after UM1 and before DM
- ...
- DM -> Processed last after all updates
```

**Data:**

```json
{
  "create_message": {
    "type": "CREATE",
    "entity_id": "12345"
  },
  "update_message_1": {
    "type": "UPDATE",
    "entity_id": "12345"
  },
  "delete_message": {
    "type": "DELETE",
    "entity_id": "12345"
  }
}
```

**Flow:**

1. Create Message
2. Consumer
3. Update Message(s)
4. Consumer
5. Delete Message
6. Consumer

