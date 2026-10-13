---
type: "practice"
difficulty: "Intermediate"
---

# How does SQS FIFO use MessageGroupId and MessageDeduplicationId to give ordering and dedup?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Visibility Timeouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Visibility_Timeouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

SQS FIFO (First-In-First-Out) ensures ordered delivery of messages within a single batch by leveraging MessageGroupId and MessageDeduplicationId. This combination allows for both deduplication and ordering, ensuring that messages are processed in the order they were sent. Let's dive into how these mechanisms work.

#### Triage
Understanding the Question

The question is asking about the specific mechanisms used by Amazon SQS FIFO to ensure message ordering and deduplication. The interviewer might be probing your understanding of message queuing principles, the differences between standard and FIFO queues, and how AWS implements these features under the hood.

#### First-Principles Solution

MessageGroupId: This is a unique identifier per batch that groups messages together. Messages within the same MessageGroupId are processed in strict order, ensuring that no out-of-order processing occurs.

MessageDeduplicationId: This ensures that duplicate messages are not delivered more than once within the same MessageGroupId. Each message must have a unique ID to prevent duplication.

Step-by-Step Explanation

Initialization:

When you send a batch of messages, each message in the batch should be assigned the same MessageGroupId.
The MessageDeduplicationId is used as a unique identifier for each individual message within that group.

Sending Messages:

You send a set of messages to an SQS FIFO queue with the same MessageGroupId. For example, you might have 5 messages in one batch.
Each of these messages will be tagged with a unique MessageDeduplicationId.

SQS Processing:

When the queue processes the messages, it checks each message's MessageDeduplicationId.
If a new message arrives with an already seen MessageDeduplicationId within the same MessageGroupId, it will be treated as a duplicate and not processed.
Messages are then placed in a FIFO manner based on their order of arrival.

Ordering Mechanism:

The queue maintains a timestamp for each message, which is used to determine the processing order.
This ensures that if any messages get delayed due to network issues or other delays, they will be processed in the correct sequence once they arrive.

#### Closing Summary

In summary, MessageGroupId and MessageDeduplicationId are crucial for ensuring both ordering and deduplication within an SQS FIFO queue. The former groups related messages together, while the latter ensures that no duplicates are processed more than once. This mechanism guarantees that messages are processed in the exact order they were sent.

#### Common Mistakes/Follow-Ups
Common Mistake: Forgetting to use a consistent MessageGroupId for all messages within a batch can lead to processing errors.

#### Follow-Up
Question: How would you handle scenarios where messages with the same MessageDeduplicationId are sent from different producers? Would this affect FIFO behavior?

To mitigate such issues, it's important to ensure that your application logic correctly handles message IDs and groups.

**Comparison:**

```
Message Group ID	Message Deduplication ID	Message Body
batch-1	id1	msg1
batch-1	id2	msg2
Response for send_message_batch
Successful:
- Id: 0, MessageId: amz.id.1
- Id: 1, MessageId: amz.id.2
Failed:
- None
```

**Code:**

```python
# Example Python code snippet for sending a batch of messages
import boto3

sqs = boto3.client('sqs')

message_group_id = 'batch-1'
messages = [
    {'MessageBody': 'msg1', 'MessageDeduplicationId': 'id1'},
    {'MessageBody': 'msg2', 'MessageDeduplicationId': 'id2'}
]

response = sqs.send_message_batch(
    QueueUrl='https://queue-url.amazonaws.com/your-queue-name',
    Entries=[
        {
            'Id': str(i),  # Unique identifier for each message
            'MessageBody': msg['MessageBody'],
            'MessageGroupId': message_group_id,
            'MessageDeduplicationId': msg['MessageDeduplicationId']
        }
        for i, msg in enumerate(messages)
    ]
)
print(response)
```

**Example input / output:**

```text
# Example input -> output for send_message_batch function
Input:
{
    'QueueUrl': 'https://queue-url.amazonaws.com/your-queue-name',
    'Entries': [
        {
            'Id': '0',
            'MessageBody': 'msg1',
            'MessageGroupId': 'batch-1',
            'MessageDeduplicationId': 'id1'
        },
        {
            'Id': '1',
            'MessageBody': 'msg2',
            'MessageGroupId': 'batch-1',
            'MessageDeduplicationId': 'id2'
        }
    ]
}

Output:
{
    "Successful": [
        {"Id": "0", "MessageId": "amz.id.1"},
        {"Id": "1", "MessageId": "amz.id.2"}
    ],
    "Failed": []
}
```

**Data:**

```json
{
  "QueueUrl": "https://queue-url.amazonaws.com/your-queue-name",
  "Entries": [
    {
      "Id": "0",
      "MessageBody": "msg1",
      "MessageGroupId": "batch-1",
      "MessageDeduplicationId": "id1"
    },
    {
      "Id": "1",
      "MessageBody": "msg2",
      "MessageGroupId": "batch-1",
      "MessageDeduplicationId": "id2"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Initiate Batch Send
4. Is MessageGroupId Same?
5. Assign UniqueMessageDeduplicationId
6. SQS Enqueue withMessageDeduplicationId
7. Process Message in FIFO Order
8. Handle Error: Messages notbatched correctly

