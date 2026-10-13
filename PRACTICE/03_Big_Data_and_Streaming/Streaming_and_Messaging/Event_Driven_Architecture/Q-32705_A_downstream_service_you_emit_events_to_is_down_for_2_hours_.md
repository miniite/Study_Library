---
type: "practice"
difficulty: "Intermediate"
---

# A downstream service you emit events to is down for 2 hours during a deploy. You use SQS. What settings ensure nothing is lost and nothing is prematurely dead-lettered, and how do you drain the backlog after recovery?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Visibility Timeouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Visibility_Timeouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To handle an outage in downstream services using AWS Simple Queue Service (SQS) effectively, we need to configure SQS to ensure that messages are not lost and do not get prematurely dead-lettered during a deployment. After the service recovers, we'll implement a mechanism to drain the backlog of messages from the queue.

#### Triaging the Question

The core issue here is ensuring message reliability during and after an outage in downstream services. The primary concerns are:

Message Delivery: Ensuring that no messages are lost.
Dead-Letter Handling: Avoiding premature dead-lettering, where a message is moved to a dead-letter queue before it should be.

#### First-Principles Solution
Configuration Settings
Visibility Timeout:
Set the visibility timeout to a higher value (e.g., 30 minutes) to ensure that messages remain in the queue longer and are not accidentally deleted or processed by other consumers.
Receive Message Wait Time Seconds:
Increase this setting during the deployment window to allow more time for messages to be received, ensuring they are processed before being automatically removed from the queue.
Dead-Letter Queue (DLQ) Configuration
Maximum Receive Count:
Set a reasonable limit on how many times a message can be received and processed by consumers.
Redrive Policy:
Configure the redrive policy to move messages back into the main queue after they've been in the DLQ for a certain period, allowing them to be processed again.
Draining Backlog After Recovery
Polling Consumers:
Implement long-polling consumers that remain connected to the queue until all messages have been consumed.
Batch Processing:
Process messages in batches to ensure efficient consumption and reduce processing load on the service.
Dead-Letter Queue Management:
Set up a script or scheduled task to drain the DLQ by re-processing messages after recovery.
Detailed Solution
Visibility Timeout

Setting the visibility timeout to 30 minutes ensures that messages remain in the queue for an extended period, reducing the risk of premature processing. This is especially important during a deployment window when services might be temporarily unavailable or undergoing maintenance.

PYTHON
import boto3

sqs = boto3.client('sqs')
queue_url = 'https://sqs.region.amazonaws.com/your-account-id/your-queue-name'

# Set visibility timeout to 30 minutes
response = sqs.set_queue_attributes(
    QueueUrl=queue_url,
    Attributes={
        'VisibilityTimeout': '1800'  # 30 minutes in seconds
    }
)
Receive Message Wait Time Seconds

Increasing the ReceiveMessageWaitTimeSeconds to a higher value (e.g., 60) during deployment allows consumers more time to process messages, reducing the likelihood of messages being removed prematurely.

PYTHON
response = sqs.receive_message(
    QueueUrl=queue_url,
    AttributeNames=['All'],
    MessageAttributeNames=['All'],
    MaxNumberOfMessages=10,
    WaitTimeSeconds=60  # Increased wait time during deployment
)
Dead-Letter Queue (DLQ) Configuration

Configure the DLQ to handle messages that are not processed within a certain number of attempts, ensuring they can be reprocessed later.

JSON
{
  "RedrivePolicy": {
    "deadLetterTargetArn": "<your-DLQ-ARN>",
    "maxReceiveCount": 10
  }
}
Draining Backlog After Recovery

After the service is recovered, use long-polling consumers to drain the backlog. Implement a script or scheduled task to monitor and process messages from both the main queue and the DLQ.

PYTHON
import boto3

sqs = boto3.client('sqs')
queue_url = 'https://sqs.region.amazonaws.com/your-account-id/your-queue-name'
dlq_url = 'https://sqs.region.amazonaws.com/your-account-id/dlq-queue-name'

def process_queue_messages(queue_url):
    response = sqs.receive_message(
        QueueUrl=queue_url,
        AttributeNames=['All'],
        MessageAttributeNames=['All'],
        MaxNumberOfMessages=10,
        WaitTimeSeconds=20  # Use lower wait time after recovery
    )
    messages = response.get('Messages', [])
    for message in messages:
        # Process the message
        process_message(message)
        # Delete the processed message from the queue
        sqs.delete_message(
            QueueUrl=queue_url,
            ReceiptHandle=message['ReceiptHandle']
        )

def drain_backlog():
    while True:
        process_queue_messages(queue_url)
        if not messages:
            break

# Example of processing a message
def process_message(message):
    # Extract and process the message payload
    print(f"Processing message: {message}")

if __name__ == "__main__":
    drain_backlog()

#### Closing Summary
+ Common Mistakes/Follow-Ups

By configuring SQS with appropriate visibility timeouts, receiving messages with increased wait times during deployments, and setting up a DLQ for message reprocessing after recovery, we can ensure that no messages are lost or prematurely dead-lettered. After the service recovers, using long-polling consumers to drain the backlog ensures all messages are processed.

#### Common mistakes
include:

Not setting appropriate visibility timeouts, leading to premature message processing.
Neglecting DLQ configuration, resulting in unrecoverable messages.
Failing to implement a robust mechanism for draining backlogs after recovery.

For follow-ups, discuss how to integrate these settings into deployment scripts and continuously monitor queue health to prevent similar issues in the future.

**Code:**

```

import boto3

sqs = boto3.client('sqs') queue_url = 'https://sqs.region.amazonaws.com/your-account-id/your-queue-name'

Set visibility timeout to 30 minutes

response = sqs.set_queue_attributes( QueueUrl=queue_url, Attributes={ 'VisibilityTimeout': '1800' # 30 minutes in seconds } )

response = sqs.receive_message( QueueUrl=queue_url, AttributeNames=['All'], MessageAttributeNames=['All'], MaxNumberOfMessages=10, WaitTimeSeconds=60 # Increased wait time during deployment )

CODE
Copy

```python
import boto3

sqs = boto3.client('sqs')
queue_url = 'https://sqs.region.amazonaws.com/your-account-id/your-queue-name'
dlq_url = 'https://sqs.region.amazonaws.com/your-account-id/dlq-queue-name'

def process_queue_messages(queue_url):
    response = sqs.receive_message(
        QueueUrl=queue_url,
        AttributeNames=['All'],
        MessageAttributeNames=['All'],
        MaxNumberOfMessages=10,
        WaitTimeSeconds=20  # Use lower wait time after recovery
    )
    messages = response.get('Messages', [])
    for message in messages:
        process_message(message)
        sqs.delete_message(
            QueueUrl=queue_url,
            ReceiptHandle=message['ReceiptHandle']
        )

def drain_backlog():
    while True:
        process_queue_messages(queue_url)
        if not messages:
            break

def process_message(message):
    print(f"Processing message: {message}")

if __name__ == "__main__":
    drain_backlog()
```

**Data:**

```json
{
  "RedrivePolicy": {
    "deadLetterTargetArn": "<your-DLQ-ARN>",
    "maxReceiveCount": 10
  }
}
```

