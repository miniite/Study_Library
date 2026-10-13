---
type: "practice"
difficulty: "Deep"
---

# What is a fan-out pattern using SNS and SQS?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Visibility Timeouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Visibility_Timeouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The fan-out pattern in an event-driven architecture, specifically when utilizing AWS Simple Notification Service (SNS) and Amazon Simple Queue Service (SQS), allows you to efficiently distribute messages from a single point of origin to multiple subscribers or workers. This pattern is particularly powerful for ensuring that different services or functions can handle the same message independently, without requiring direct communication between them.

#### Triaging the Question

The question asks us to explain the fan-out pattern in an event-driven architecture using AWS SNS and SQS. The core of this pattern involves:

Understanding how messages are published from a single source (the publisher) to multiple subscribers.
Exploring the role of SNS as the central intermediary that routes the message based on subscription patterns.
Investigating how SQS acts as an endpoint for these subscribers, ensuring each receives its own copy of the message.

#### First-Principles Solution

Publisher Publishes a Message:

A single entity (publisher) sends a message to SNS. This can be triggered by various events or directly by the application.

SNS Distributes Messages:

SNS acts as a broker, distributing the message to all subscribers based on predefined subscription rules.
These rules can include topics, protocols (HTTP/HTTPS, email), and other conditions.

Subscribers Consume Messages:

Each subscriber receives its own copy of the message from SNS.
Subscribers are implemented as SQS queues or AWS Lambda functions that listen to specific topics in SNS.

Independent Processing:

The subscribers can process messages independently, ensuring loose coupling and high scalability.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the fan-out pattern using SNS and SQS enables efficient message distribution by decoupling the publisher from the subscribers. Each subscriber can act on the same event independently, making the architecture more robust and scalable. Common mistakes include not properly configuring SNS topics or neglecting to set up appropriate error handling for failed messages.

#### Follow-ups
might involve discussing:

Best practices for setting up SNS/SQS configurations.
Error handling mechanisms in SQS.
Security considerations when using these services.
Performance optimization techniques.

**Comparison:**

```
Publisher	Action	SNS	Subscribers
	Publishes a message	Distributes messages
	to SNS Topic	to all subscribed queues
Subscriber (SQS)	Action	Outcome
	Receives and processes	Processes the message independently
	a copy of the message
```

**Code:**

```
Example Python code snippet for publishing a message to SNS

import boto3

sns_client = boto3.client('sns') response = sns_client.publish( TopicArn='arn:aws:sns:us-west-2:123456789012:my-topic', Message='Hello, this is a test message!' ) print(response)

CODE
Copy

```python
# Example Python code snippet for subscribing to an SNS topic via SQS queue
import boto3

sqs_client = boto3.client('sqs')
response = sqs_client.create_queue(QueueName='my-queue', Attributes={'DelaySeconds': '5'})
queue_url = response['QueueUrl']
sns_client.subscribe(
    TopicArn='arn:aws:sns:us-west-2:123456789012:my-topic',
    Protocol='sqs',
    Endpoint=queue_url
)
```

**Example input / output:**

```text
Input:
Publisher publishes a message to SNS Topic ARN.

Output:
Message is distributed to all subscribers (SQS queues) associated with the topic.
Each subscriber consumes and processes the message independently.
```

**Data:**

```json
{
  "SNS Topic ARN": "arn:aws:sns:us-west-2:123456789012:my-topic",
  "SQS Queue URL": "https://sqs.us-west-2.amazonaws.com/123456789012/my-queue"
}
```

**Flow:**

1. Publisher
2. Publish Message
3. SNS
4. All Subscribers (SQS)
5. Queue 1
6. Queue 2
7. Process Message
8. Process Message

