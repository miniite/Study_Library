---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between AWS SQS standard and FIFO queues?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Visibility Timeouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Visibility_Timeouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When comparing Amazon Simple Queue Service (SQS) Standard queues to FIFO queues, it's important to understand that these two types of queues serve fundamentally different purposes in your event-driven architecture. SQS Standard is designed for throughput and scalability, while FIFO queues offer strict ordering and exactly-once processing guarantees.

First, let's break down what each type of queue does:

SQS Standard Queues: These are ideal for scenarios where message order isn't critical but you need high-throughput and low-latency. Messages can be sent out of order and duplicates might occur.
SQS FIFO Queues: These guarantee that messages are processed in the exact order they were received, ensuring exactly-once processing semantics.

Now, let's dive into the key differences and their implications:

Message Ordering:

Standard Queues: Messages can be delivered to consumers out of the order they were sent. This makes them suitable for scenarios where you don't need strict ordering.
FIFO Queues: Each message has a unique deduplication ID, ensuring that messages are processed in the exact order they arrived. This is crucial for financial transactions, command processing, or any scenario requiring strict order.

Message Visibility and Delivery:

In Standard Queues, visibility timeouts can lead to multiple consumers receiving the same message if it's not deleted before its timeout period ends.
In FIFO Queues, visibility timeouts are managed automatically by AWS to ensure no duplicates are processed until they're removed.

Latency and Throughput:

Standard queues generally offer lower latency compared to FIFO queues because of their design for high throughput and reduced guarantees around order and duplicate handling.
FIFO queues have higher latencies due to the additional overhead of maintaining message ordering and deduplication.

Cost and Complexity:

Using Standard Queues is usually more cost-effective, as they are designed for simpler use cases where exact order isn't essential.
FIFO Queues come with increased costs and complexity but offer the necessary guarantees for specific business requirements.

Scalability and Performance:

Standard queues can scale better in scenarios where you don't need message ordering because they handle messages more efficiently.
FIFO queues might require careful configuration and monitoring to ensure optimal performance, especially under high load conditions.

In summary, the choice between SQS Standard and FIFO queues depends on your specific needs. If strict order and exactly-once processing are critical, FIFO is the way to go. However, if you prioritize lower latency and higher throughput with less stringent ordering requirements, then Standard Queues might be more appropriate.

**Comparison:**

```
Queue Type	Ordering Guarantee	Duplicate Handling	Latency	Throughput
Standard	No	None	Lower	Higher
FIFO	Yes	Strict	Higher	Lower
```

**Code:**

```
Example of sending a message to an SQS Standard queue (pseudo-code)

import boto3

sqs = boto3.client('sqs') queue_url = 'https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue'

response = sqs.send_message( QueueUrl=queue_url, MessageBody='Hello, this is a standard queue message.' ) print(response['MessageId'])

CODE
Copy

```python
# Example of sending a message to an SQS FIFO queue (pseudo-code)
import boto3

sqs = boto3.client('sqs')
queue_url = 'https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo'

response = sqs.send_message(
    QueueUrl=queue_url,
    MessageBody='Hello, this is a FIFO queue message.',
    MessageGroupId='group1'
)
print(response['MessageId'])
```

**Example input / output:**

```

Input (Standard Queue): { "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue", "MessageBody": "Hello, this is a standard queue message." }

Output: { "MessageId": "mymessageid123" }

CODE
Copy

```plaintext
Input (FIFO Queue):
{
  "QueueUrl": "https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo",
  "MessageBody": "Hello, this is a FIFO queue message.",
  "MessageGroupId": "group1"
}

Output:
{
  "MessageId": "mymessageid123"
}
```

**Data:**

```json
{
  "StandardQueue": {
    "Url": "https://sqs.us-east-1.amazonaws.com/123456789012/my-standard-queue",
    "MessagesSent": [
      { "MessageBody": "Hello, this is a standard queue message.", "Timestamp": "2023-10-01T12:00:00Z" }
    ]
  },
  "FIFOQueue": {
    "Url": "https://sqs.us-east-1.amazonaws.com/123456789012/my-fifo-queue.fifo",
    "MessagesSent": [
      { "MessageBody": "Hello, this is a FIFO queue message.", "MessageGroupId": "group1", "Timestamp": "2023-10-01T12:05:00Z" }
    ]
  }
}
```

**Flow:**

1. Standard Queue
2. Send Message
3. Deliver to Consumer(OutOfOrder)
4. Consumer Process
5. FIFO Queue
6. Send Message
7. Queue Ensures Order
8. Deliver to Consumer (InOrder)
9. Consumer Process

