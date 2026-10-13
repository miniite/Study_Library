---
type: "practice"
difficulty: "Expert"
---

# Design a streaming pipeline that must interoperate with an external partner's Kafka you don't control and whose schema changes without notice. How do you build a defensive boundary so their instability can't corrupt your internal streams?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Consuming Untrusted Streams](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Streaming_Architecture/Provisional_and_Final_Results/Consuming_Untrusted_Streams.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a real-time streaming pipeline that needs to interoperate with an external partner's Kafka cluster, which they don't control and whose schema might change without notice, we must build a robust boundary to protect our internal streams from potential instability or data corruption. The core idea is to decouple the interface between our system and the external Kafka topic through a series of intermediate components that can handle schema changes and ensure data integrity.

#### Triage
The question revolves around creating a resilient design for interfacing with an uncontrolled external Kafka stream, focusing on handling schema changes without affecting internal processes. We need to establish robust boundaries to manage incoming data streams effectively, ensuring that any issues in the external system do not propagate into our critical systems.

#### Assumptions

The external Kafka topic is managed by a third-party partner who may introduce schema or data format changes.
Our system requires real-time processing of this stream but must maintain reliability and integrity.
We have some control over how we consume from the Kafka topic, allowing us to implement custom logic for handling data.

#### First-principles solution

Kafka Consumer: Consume messages from the external Kafka topic using a resilient consumer group strategy. This ensures that if the partner's system is unstable, our consumer can failover gracefully.
Message Parsing and Validation: Implement a message parsing layer to validate incoming messages against known schemas or a generic schema validation tool. This helps catch malformed data early in the pipeline.
Event Sourcing: Store raw Kafka messages in an event store (e.g., another Kafka topic) before processing them. This provides a persistent history of all ingested data and allows us to replay events if needed.
Schema Evolution Handling: Develop a mechanism for handling schema changes by versioning schemas or using dynamic schema mapping tools like Apache Avro or Protobuf.
Transformation and Processing: Process the validated and stored messages in your internal stream processing system, ensuring that any downstream systems only interact with sanitized data.

#### Closing summary
By implementing these layers of defense, we can maintain a stable and reliable internal streaming architecture despite external instability. Key takeaways include robust schema handling, event sourcing for persistence, and a well-defined boundary between our systems and the external Kafka topic.

#### Common mistakes/follow-ups

Failing to implement proper validation and parsing logic, leading to corrupted internal data.
Not using an event store, making it difficult to recover from failures or validate historical data.
Ignoring schema versioning in dynamic environments, which can lead to breaking changes in processing logic.
Overlooking the need for failover strategies that ensure continuous service despite external system issues.

**Comparison:**

```
Consumer Group	Kafka Topic	Role
my-group	external-topic	Consume and validate messages
```

**Code:**

```python
# Example Python code for Kafka consumer and validation
from confluent_kafka import Consumer, KafkaException, KafkaError
import avro.schema
from avro.datafile import DataFileReader
from avro.io import DatumReader

# Kafka consumer setup
consumer = Consumer({
    'bootstrap.servers': 'kafka-broker:9092',
    'group.id': 'my-group'
})

def consume_kafka():
    try:
        consumer.subscribe(['external-topic'])
        while True:
            msg = consumer.poll(1.0)
            if msg is None:
                continue
            if msg.error():
                raise KafkaException(msg.error())
            else:
                value_schema_str = """{
                    "type": "record",
                    "name": "User",
                    "fields": [
                        {"name": "id", "type": "int"},
                        {"name": "name", "type": "string"}
                    ]
                }"""
                schema = avro.schema.Parse(value_schema_str)
                reader = DataFileReader(msg.value(), DatumReader())
                for user in reader:
                    print(user)
    finally:
        consumer.close()

consume_kafka()
```

**Example input / output:**

```

Example input:

JSON
Copy
{"id": 1, "name": "John Doe"}

Example output:

CODE
Copy
{'id': 1, 'name': 'John Doe'}
```

**Data:**

```json
{
  "value_schema": {
    "type": "record",
    "name": "User",
    "fields": [
      {"name": "id", "type": "int"},
      {"name": "name", "type": "string"}
    ]
  }
}
```

**Flow:**

1. External Kafka Topic
2. Kafka Consumer Group
3. Parsing and Validation Layer
4. Event Sourcing (Kafka)
5. Internal Processing Pipeline

