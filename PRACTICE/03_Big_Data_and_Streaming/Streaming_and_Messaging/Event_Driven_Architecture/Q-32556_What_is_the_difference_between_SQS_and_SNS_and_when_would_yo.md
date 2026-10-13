---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between SQS and SNS, and when would you use them together?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Visibility Timeouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Visibility_Timeouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between Amazon Simple Queue Service (SQS) and Amazon Simple Notification Service (SNS), we need to consider their fundamental purposes and how they can be used together in an event-driven architecture.

TL;DR:

SQS is designed for asynchronous, reliable messaging where a producer sends messages to a queue, which are then consumed by one or more workers. It’s ideal for handling batch processing, decoupling components, and managing load.
SNS is used for push notifications and can also act as a pub/sub pattern broker where publishers send messages to multiple subscribers asynchronously. SNS can integrate with SQS queues, allowing it to trigger actions in response to events.

#### Triage
The question is probing your understanding of AWS's messaging services and their roles in an event-driven architecture. The assumptions are that you're familiar with both services and their use cases. We'll start by explaining each service individually before discussing how they can be used together effectively.

#### First-Principles Solution

SQS (Simple Queue Service):

Purpose: SQS is an asynchronous messaging system where producers send messages to a queue, which are then consumed by one or more workers.
Use Cases:
Batch processing and background tasks.
Decoupling components of your application to reduce dependencies.
Managing load spikes by batching requests.
Ensuring data consistency when performing operations on multiple services.

SNS (Simple Notification Service):

Purpose: SNS is a pub/sub pattern broker that allows publishers to send messages to multiple subscribers asynchronously via HTTP, email, SMS, or Lambda functions.
Use Cases:
Push notifications to mobile devices.
Sending alerts and status updates.
Triggering actions in response to events, such as triggering SQS queues.

Using SQS with SNS Together:

You can use SNS to trigger an SQS queue, which is particularly useful when you want to decouple the source of your event from the consumer.
For example, if a new record is created in a database, you could have SNS publish this information to an SQS queue. Then, your workers can consume messages from that queue and perform necessary actions.

#### Example Scenario

Let's consider a scenario where you're building a real-time notification system for users when they receive a message. Here’s how you might use both services together:

Event Generation:

A user sends a message to another user.
This event is captured and sent as a message through SNS.

Notification Delivery:

The message published by SNS can trigger an SQS queue.
An SQS worker then processes the message and sends a notification to the recipient via email, SMS, or any other delivery method.

#### Closing Summary

Using both SNS and SQS together provides a flexible and robust event-driven architecture. SNS handles the broadcasting of events to multiple subscribers, while SQS ensures that these events are processed asynchronously and can be handled efficiently by your application workers.

#### Common Mistakes/Follow-Ups
Mistake: Overusing SNS for heavy processing tasks; SNS is not designed for handling complex logic or batch processing.

#### Follow-Up
When would you choose one over the other? For simple event notifications, use SNS. For more complex message handling and batching, use SQS.

**Comparison:**

```
Service	Purpose	Use Case
SQS	Async messaging for producers & consumers	Batch processing, decoupling components
SNS	Pub/sub pattern broker for notifications	Push notifications, alerting systems
```

**Code:**

```python
# Example of publishing a message to an SQS queue via SNS in Python

import boto3

def publish_to_sqs_via_sns(topic_arn, queue_url):
    sns_client = boto3.client('sns')
    sqs_client = boto3.client('sqs')

    # Publish the message to SNS
    response = sns_client.publish(
        TopicArn=topic_arn,
        Message='This is a test message',
        Subject='Test Message'
    )

    # Extract the message ID from the SNS response
    message_id = response['MessageId']

    # Send a message to SQS using the message ID from SNS
    sqs_client.send_message(
        QueueUrl=queue_url,
        MessageAttributes={
            'Subject': {
                'DataType': 'String',
                'StringValue': 'Test Message'
            }
        },
        MessageBody=(
            f'Notification triggered by SNS with message ID: {message_id}'
        )
    )

publish_to_sqs_via_sns('arn:aws:sns:us-west-2:123456789012:my-topic', 'https://sqs.us-west-2.amazonaws.com/123456789012/my-queue')
```

**Example input / output:**

```text
Input:
- SNS Topic ARN: arn:aws:sns:us-west-2:123456789012:my-topic
- SQS Queue URL: https://sqs.us-west-2.amazonaws.com/123456789012/my-queue

Output:
1. SNS publishes a message with the subject "Test Message" and body "This is a test message".
2. The message ID from the SNS response is used to trigger an SQS queue.
3. An SQS message is sent with attributes and body referencing the SNS message.
```

**Data:**

```json
{
  "topic_arn": "arn:aws:sns:us-west-2:123456789012:my-topic",
  "queue_url": "https://sqs.us-west-2.amazonaws.com/123456789012/my-queue"
}
```

**Flow:**

1. User sends message
2. SNS publishes event to SQSqueue
3. Worker consumes from SQS
4. Process message and sendnotification

