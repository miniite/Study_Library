---
type: "practice"
difficulty: "Deep"
---

# Your remediation runbook consumer must not fire the same 'scale-up' action twice when the same anomaly is reported by two mirrored clusters during failover. How do you dedupe?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent and Transactional Producers](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Idempotent_and_Transactional_Producers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that our remediation runbook consumer does not fire the same "scale-up" action twice when the same anomaly is reported by two mirrored clusters during a failover, we need to implement a mechanism for deduplication. At an AIOps 2/3 level, this involves understanding how Kafka works, designing a robust processing pipeline, and considering edge cases.

Kafka is a distributed streaming platform that enables us to handle large volumes of messages in real-time. In our scenario, each mirrored cluster will produce anomaly reports as Kafka topics. The consumer needs to process these topics without duplicating actions for the same anomaly.

The key idea here is to use a combination of state management and unique identifiers to ensure that only one "scale-up" action is fired per unique anomaly event. Let’s break down the solution from first principles.

**Comparison:**

| Anomaly ID | Cluster 1 | Cluster 2 |
|------------|-----------|-----------|
| A123       | Processed | Not Yet  |
| B456       | Not Yet  | Not Yet  |

**Code:**

```python
from kafka import KafkaConsumer
import uuid

# Define the Kafka consumer configuration
consumer = KafkaConsumer(
    'anomaly-topic',
    bootstrap_servers=['localhost:9092'],
    auto_offset_reset='earliest',
    enable_auto_commit=True,
    group_id='scale-up-group'
)

# Unique identifier for each processing cycle
session_token = str(uuid.uuid4())

def process_message(message):
    # Extract the anomaly ID from the message
    anomaly_id = message.value.decode('utf-8')

    if not is_deduplicated(anomaly_id, session_token):
        # Deduplicate check logic
        handle_scale_up_action(anomaly_id)
        mark_as_processed(anomaly_id)

def is_deduplicated(anomaly_id, session_token):
    # Check the state store to see if this anomaly ID has been processed in this session
    return not get_state(anomaly_id, session_token)

def handle_scale_up_action(anomaly_id):
    # Logic for firing the scale-up action
    print(f"Scaling up for anomaly: {anomaly_id}")

def mark_as_processed(anomaly_id):
    # Update state store to mark the anomaly as processed in this session
    set_state(anomaly_id, session_token)

# Main loop to process messages from Kafka topic
for message in consumer:
    process_message(message)
```

**Example input / output:**

```text
Sample input and output:

- Input: A Kafka message with `anomaly-id` "A123" from one mirrored cluster.
- Output: The remediation runbook consumer processes the message and prints "Scaling up for anomaly: A123".

- Input: Another Kafka message with the same `anomaly-id` "A123" from a different mirrored cluster.
- Output: Deduplication mechanism checks, identifies it as processed in this session, no action taken.

- Input: A new anomaly report with `anomaly-id` "B456".
- Output: The remediation runbook consumer processes the message and prints "Scaling up for anomaly: B456".
```

**Data:**

```yaml
# Example Kafka topic schema for anomaly reports
anomaly-report:
  type: object
  properties:
    anomaly-id:
      type: string
    cluster-name:
      type: string
```

**Flow:**

1. Yes
2. No
3. Receive Anomaly Report
4. Is Anomaly Deduplicated?
5. Skip Action
6. Handle Scale-Up Action
7. Mark Anomaly as Processed

