---
type: "practice"
difficulty: "Deep"
---

# Your streaming pipeline must apply GDPR-style deletion to user-tagged telemetry that's sitting in a Kafka log for replay. How do you honor deletion in an append-only log used for anomaly retraining?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of honoring GDPR-style deletion in a streaming pipeline using Kafka, we need to design a system that can selectively remove or anonymize user-tagged telemetry data while preserving the integrity and usability of the log for anomaly retraining purposes. The key is to implement a mechanism that allows for both efficient data removal and seamless retraining, without compromising the stream's append-only nature.

This problem involves several layers of complexity:

Understanding GDPR Requirements: GDPR requires organizations to ensure that personal data can be deleted promptly when requested by users.
Kafka's Append-Only Nature: Kafka is designed for exactly-once delivery and event sourcing, making traditional deletion methods challenging.
Retraining Anomalies: The log must support retraining models with updated or anonymized data.

We will break down the solution step-by-step from first principles to ensure a robust implementation.

**Comparison:**

```
Step	Action	Description
1	Extract Metadata	Parse the message to extract user ID and other relevant data.
2	Check Deletion Request	Determine if the user's data needs to be deleted or anonymized based on a request or policy.
3	Anonymize Data	Modify the telemetry data as required (e.g., replace PII with 'ANONYMIZED').
4	Retain for Training	Ensure the modified message is retained in the Kafka log for retraining purposes.
```

**Code:**

```python
# Example Python code for Kafka message processing and deletion
from kafka import KafkaConsumer, KafkaProducer
import json

def process_message(message):
    # Extract metadata from the message (e.g., user ID)
    metadata = json.loads(message.value)
    user_id = metadata['user_id']

    if is_user_data_deletable(user_id):  # Function to check deletion request
        return anonymize_message(message)  # Anonymize and return modified message

    return message  # Return original message for retraining

def is_user_data_deletable(user_id):
    # Placeholder function that would interface with a data governance system
    pass

def anonymize_message(message):
    # Placeholder function to modify the message (e.g., remove user ID, replace PII)
    modified_message = json.loads(message.value)
    modified_message['user_id'] = 'ANONYMIZED'
    return json.dumps(modified_message).encode('utf-8')
```

**Example input / output:**

```

Input Message:

JSON
Copy
{
  "timestamp": 1633072800,
  "user_id": "12345",
  "telemetry_data": {
    "metric1": 42.5,
    "metric2": 60.3
  }
}

Output Message (after processing):

JSON
Copy
{
  "timestamp": 1633072800,
  "user_id": "ANONYMIZED",
  "telemetry_data": {
    "metric1": 42.5,
    "metric2": 60.3
  }
}
```

**Data:**

```json
{
  "message": {
    "timestamp": 1633072800,
    "user_id": "12345",
    "telemetry_data": {
      "metric1": 42.5,
      "metric2": 60.3
    }
  }
}
```

**Flow:**

1. Yes
2. No
3. Message Received
4. Is User Data Deletable?
5. Anonymize Message
6. Retain in Kafka Log forRetraining
7. Return Original Message toKafka Log
8. Stream Continues

