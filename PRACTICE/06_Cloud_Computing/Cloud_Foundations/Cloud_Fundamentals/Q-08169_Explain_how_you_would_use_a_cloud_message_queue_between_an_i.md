---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you would use a cloud message queue between an ingestion service and a feature computation worker so bursty upstream data does not overwhelm the workers.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Sync vs Async Processing](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Sync_vs_Async_Processing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with bursty data in a system where we need to process feature computations, using a cloud message queue can help manage the load effectively. In this scenario, an ingestion service would send messages (data chunks) into the queue, and worker services would consume these messages at their own pace. This setup prevents the workers from being overwhelmed by bursts of data.

To ensure that bursty upstream data does not overwhelm the feature computation workers, we need to design a system that can handle variable loads efficiently. Here’s how I would approach this:

Understand the Problem: The ingestion service sends unpredictable amounts of data at irregular intervals. We want to smooth out this load without dropping any messages.

Identify Key Components:

Ingestion Service: Sends data to the queue.
Message Queue (e.g., AWS SQS, Google Cloud Pub/Sub): Manages the buffer between the ingestion and processing services.
Feature Computation Workers: Consume messages from the queue for processing.

Queue Mechanism: A message queue acts as a buffer that can hold incoming data until it is processed. This helps in absorbing bursts of data without causing immediate load on the workers.

Scalability and Load Balancing:

The number of worker services can be scaled automatically based on the current load.
Queues support FIFO (First-In-First-Out) processing, ensuring that messages are processed in the order they were received.

Retries and Dead Letter Queues: In case a message fails to process, it should be put into a dead letter queue where it can be reviewed or retried.

Message Prioritization: If certain types of data require higher priority processing, we can use features like prioritized messages in the queue.

Monitoring and Tuning: Regularly monitor the health and performance of the system to ensure that it is handling bursts efficiently. Adjust capacity planning based on historical patterns if necessary.

**Comparison:**

```
Queue Type	AWS SQS	Google Cloud Pub/Sub
Message Retention	Yes	Yes
Dead Letter Queue	Supported	Supported
Prioritization	Yes	Yes
```

**Code:**

```
Example Python code snippet for producing messages to a message queue using AWS SDK

import boto3 from botocore.exceptions import ClientError

def send_message_to_queue(queue_url, message_body): sqs = boto3.client('sqs') try: response = sqs.send_message( QueueUrl=queue_url, DelaySeconds=10, MessageAttributes={}, MessageBody=(message_body) ) except ClientError as e: print(f"Failed to send message: {e}") else: print("Message sent successfully")

CODE
Copy

```python
# Example Python code snippet for consuming messages from a queue and processing them

import boto3
from botocore.exceptions import ClientError

def process_message_from_queue(queue_url):
    sqs = boto3.client('sqs')

    try:
        # Receive message from the SQS queue
        response = sqs.receive_message(
            QueueUrl=queue_url,
            AttributeNames=['All'],
            MaxNumberOfMessages=10,
            MessageAttributeNames=['All'],
            WaitTimeSeconds=20
        )

        if 'Messages' in response:
            for msg in response['Messages']:
                # Process the message
                process_message(msg)

                # Delete the processed message from the queue
                receipt_handle = msg['ReceiptHandle']
                sqs.delete_message(
                    QueueUrl=queue_url,
                    ReceiptHandle=receipt_handle
                )
    except ClientError as e:
        print(f"Failed to receive or delete messages: {e}")
```

**Example input / output:**

```text
Input (Ingestion Service):
- Sends bursts of data messages to the queue

Output (Worker Services):
- Consume messages from the queue at their own pace, ensuring smooth processing
- Handle retries and dead-letter messages appropriately
```

**Data:**

```json
{
  "QueueConfiguration": {
    "QueueName": "ingestion-worker-queue",
    "VisibilityTimeout": 30,
    "MaxNumberOfMessages": 10,
    "ReceiveMessageWaitTimeSeconds": 20
  }
}
```

**Flow:**

1. Yes
2. No
3. Ingestion Service
4. Queue Full?
5. Send Message to Queue
6. Wait for Space in Queue
7. Message Queued
8. Worker Services
9. Process Messages at Own Pace

