---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between SNS standard and FIFO topics, and how do subscription filter policies work?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Visibility Timeouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Event_Driven_Architecture/Visibility_Timeouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In an event-driven architecture, message queues play a crucial role in decoupling services and ensuring reliable data processing. This response will explain the differences between SNS standard topics and FIFO (First-In-First-Out) topics in AWS SNS, as well as how subscription filter policies work.

#### Triaging the Question

The question is asking for the distinctions between two types of topics in Amazon Simple Notification Service (SNS): standard topics and FIFO topics. Additionally, it wants to understand how subscription filter policies operate within these contexts. The interviewer might be probing your understanding of AWS SNS features, queue-based architectures, and data processing nuances.

#### First-Principles Solution

Standard Topics vs. FIFO Topics:

Standard Topics: These are non-order-preserving topics where messages can be delivered to multiple subscribers in a best-effort manner.
FIFO Topics: These maintain the order of published messages, ensuring that messages are delivered exactly once and in the exact order they were sent.

Subscription Filter Policies:

Subscription filter policies allow you to define rules on the SNS topic side, which determine what specific data within a message should be processed by a subscription.
Detailed Explanation
Standard Topics
Ordering: Messages are not guaranteed to arrive in the order they were sent. This is fine for scenarios where reordering or duplication of messages can be handled gracefully (e.g., processing logs).
Scalability: High throughput and low latency, making them suitable for high-volume applications.
Cost: Generally cheaper due to lower overhead.
FIFO Topics
Ordering: Messages are delivered in the order they were sent, which is critical for scenarios like financial transactions or event sequencing.
Delivery Guarantees: Ensures at-least-once delivery (messages might be delivered multiple times) and exactly-once semantics with dead-letter queues.
Scalability: Can handle high volume but may introduce slight delays due to the need to maintain order.
Subscription Filter Policies
These policies allow fine-grained control over which data in a message is processed by a subscription. For instance, you might want to only process certain fields or patterns within a JSON payload.
Filter Policy Example: If your application processes financial transactions and you receive messages with multiple transaction types, you can set up filter policies to ensure that specific transaction types are sent to the correct subscribers.

#### Closing Summary

Understanding these differences is crucial for designing scalable, fault-tolerant systems. Standard topics are suitable for high-throughput scenarios where order isn’t critical, while FIFO topics provide ordered delivery which is essential in sequential processing scenarios. Subscription filter policies enhance flexibility by allowing you to tailor message processing based on specific criteria.

#### Common Mistakes
and Follow-Ups
Mistake: Assuming standard SNS topics can handle order-critical tasks.

#### Follow-Up
Highlight the importance of using FIFO topics for such cases.
Mistake: Ignoring the implications of at-least-once delivery in FIFO topics.

#### Follow-Up
Discuss how to manage deduplication and ensure idempotency.

**Comparison:**

```
Topic Type	Order Guarantee	Delivery Guarantees	Scalability
Standard	No	Best Effort	High
FIFO	Yes	At Least Once	Moderate
```

**Code:**

```python
# Example Code to Publish a Message to an SNS Topic
import boto3

sns = boto3.client('sns')

response_standard = sns.publish(
    TopicArn='arn:aws:sns:<region>:<account-id>:standard-topic',
    Message='This is a standard message.'
)

response_fifo = sns.publish(
    TopicArn='arn:aws:sns:<region>:<account-id>:fifo-topic',
    Message='This is an FIFO message.',
    MessageAttributes={
        'Ordering': {
            'DataType': 'String',
            'Value': 'high'
        }
    }
)
```

**Example input / output:**

```text
Input:
- Publish 'This is a standard message.' to standard-topic.
- Publish 'This is an FIFO message.' with Ordering attribute set to 'high' to fifo-topic.

Output (for standard topic):
- Message delivered in best-effort order.

Output (for FIFO topic):
- Message ordered and delivered as part of the queue sequence.
```

**Data:**

```json
{
  "TopicArn": "arn:aws:sns:<region>:<account-id>:standard-topic",
  "Message": "This is a standard message."
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Send Message to StandardTopic
6. Message Delivered in Order?
7. Process Message
8. Store Message for LaterProcessing
9. Send Message to FIFO Topic
10. Order Maintained?
11. Process Message in Order
12. Reprocess Messages asNecessary

