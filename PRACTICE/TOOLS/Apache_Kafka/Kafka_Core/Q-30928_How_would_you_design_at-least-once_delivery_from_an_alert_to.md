---
type: "practice"
difficulty: "Deep"
---

# How would you design at-least-once delivery from an alert topic into PagerDuty so no critical page is ever lost, even across a total consumer restart?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Topic Design for Re-sync](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Topic_Design_for_Re-sync.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure at-least-once delivery of alerts to PagerDuty, we need to design a robust system that handles failures gracefully and ensures messages are not missed. The key components involved include Kafka as our messaging broker, and PagerDuty as the external service where critical pages will be sent. We'll leverage Kafka's built-in features along with some additional measures to achieve this goal.

First, let’s reframe the question: The interviewer is probing into understanding how we can ensure message delivery reliability in a distributed system using Kafka, especially focusing on handling consumer failures and ensuring messages are not lost. We need to consider state management, idempotency, and fault tolerance mechanisms.

#### Assumptions

The alerts are published to a specific Kafka topic.

#### Cons
umers read from this topic and send the alert details to PagerDuty.
There could be multiple consumers in a consumer group to distribute load and ensure high availability.
We need to handle scenarios where a consumer crashes or restarts, ensuring that all messages are delivered at least once.
Step-by-Step Solution
Kafka Topic Configuration:
Ensure the topic is configured with appropriate retention policies to retain messages for an extended period in case of failures.

#### Cons
umer Group Management:
Use Kafka’s consumer groups to manage multiple consumers reading from a single topic.
Message Idempotency:
Ensure that the message handler at PagerDuty can handle duplicate alerts without causing any issues (e.g., sending the same alert more than once).

#### Cons
umer Failure Handling:
Implement a strategy for handling consumer failures, such as rebalancing and redelivery.
ACK Mechanism:
Use Kafka’s ACK mechanism to ensure that messages are acknowledged after being processed by consumers.
Kafka Streams or KSQLDB Integration:
Optionally, use Kafka Streams or KSQLDB for processing and managing stateful operations.
Detailed Steps

Topic Configuration:

PLAINTEXT
# Example configuration
topic_config = {
    'retention.ms': 86400000,  # Retain messages for 24 hours
    'cleanup.policy': 'delete',  # Automatically clean up old data
}

#### Cons
umer Group Setup:

PYTHON
from kafka import KafkaConsumer

#### cons
umer = KafkaConsumer('alert-topic',
                        bootstrap_servers=['localhost:9092'],
                        group_id='alert-consumer-group')
for message in consumer:
    # Process the alert and send it to PagerDuty
    process_alert(message.value)

Message Idempotency:

Ensure that the PagerDuty API can handle duplicate alerts. For example, you might check if a page with certain criteria already exists before sending another one.

#### Cons
umer Failure Handling:

PYTHON
def on_failure(exception):
    print(f'Consumer failed: {exception}')
    # Implement retry logic or log for further investigation

#### cons
umer = KafkaConsumer('alert-topic',
                        bootstrap_servers=['localhost:9092'],
                        group_id='alert-consumer-group',
                        fail_on_error=False,
                        on_message_error=on_failure)

ACK Mechanism:

Configure the consumer to acknowledge messages only after they are successfully processed and sent to PagerDuty.

State Management:

PYTHON
from kafka import KafkaProducer

producer = KafkaProducer(bootstrap_servers=['localhost:9092'])

def send_alert_to_pager_duty(alert):
    response = produce_and_send_to_pager_duty(alert)
    if not response.ok:
        # Log the failure and retry later using a backoff strategy
        pass

#### Closing Summary
+ Common Mistakes/Follow-Ups

By leveraging Kafka’s built-in features, such as consumer groups, idempotent processing, and ACK mechanisms, we can ensure that alerts are delivered to PagerDuty at least once. Key challenges include handling consumer failures gracefully and ensuring state consistency. Some common mistakes include not properly configuring topic retention policies, failing to handle duplicates appropriately, or overlooking the need for robust error handling.

For follow-ups, one might discuss additional strategies like using Kafka Connect for external system integration, implementing custom retry logic with exponential backoff, or integrating monitoring tools to track message delivery status.

**Comparison:**

| Message ID | Alert Details                  |
|------------|--------------------------------|
| 1          | Critical system down           |
| 2          | High CPU usage detected        |

**Code:**

```

from kafka import KafkaConsumer

consumer = KafkaConsumer('alert-topic', bootstrap_servers=['localhost:9092'], group_id='alert-consumer-group') for message in consumer: process_alert(message.value)

CODE
Copy

```python
def on_failure(exception):
    print(f'Consumer failed: {exception}')
    # Implement retry logic or log for further investigation

consumer = KafkaConsumer('alert-topic',
                        bootstrap_servers=['localhost:9092'],
                        group_id='alert-consumer-group',
                        fail_on_error=False,
                        on_message_error=on_failure)
```

**Example input / output:**

```text
Input:
[{"message_id": 1, "alert_details": "Critical system down"},
 {"message_id": 2, "alert_details": "High CPU usage detected"}]

Output (to PagerDuty):
- Alert ID: 1 - Critical system down
- Alert ID: 2 - High CPU usage detected
```

**Data:**

```json
{
  "topic_config": {
    "retention.ms": 86400000,
    "cleanup.policy": "delete"
  },
  "consumer_group": "alert-consumer-group",
  "alert_data": [
    {"message_id": 1, "alert_details": "Critical system down"},
    {"message_id": 2, "alert_details": "High CPU usage detected"}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Alert Topic
4. Kafka Consumer Group
5. Failure?
6. Process Alert and Send toPagerDuty
7. Handle Failure (Retry, Log)

