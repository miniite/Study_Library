---
type: "practice"
difficulty: "Deep"
---

# Design a mechanism to detect and quarantine a producer that starts sending events with a corrupt or unregistered schema before consumers are affected.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Schema Formats and Registry](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Schema_Formats_and_Registry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a mechanism to detect and quarantine producers that start sending events with a corrupt or unregistered schema, we need to ensure that the Kafka ecosystem can validate schemas in real-time and take proactive actions before any consumers are impacted. This involves integrating schema validation tools like Confluent Schema Registry with Kafka's topic mechanisms.

#### Triaging the Question

The core question is about designing a system that ensures data integrity by validating schemas at the producer level. The key aspects to probe include:

Schema Validation: Ensuring that each message adheres to the registered schema.
Real-Time Detection: Identifying producers sending corrupt or unregistered messages in near real-time.
Quarantine Mechanism: Implementing a mechanism to isolate such producers without disrupting the rest of the system.

#### Assumptions
We are using Confluent Schema Registry for managing and validating schemas.
Producers will send messages to specific Kafka topics, which are associated with registered schemas.
The system should be scalable and minimize overhead on existing producers and consumers.

#### First-Principles Solution
Schema Validation at the Producer: Integrate a validation step in the producer code that checks each message against the registered schema before sending it to Kafka. This can be done using the Confluent Schema Registry's Java or Python libraries.
Real-Time Detection and Logging: Implement logging mechanisms within the producer to log any validation failures. These logs should include details like the producer ID, topic name, timestamp of the message, and any relevant error messages.
Alerting Mechanism: Set up alerting on the Kafka cluster or a centralized monitoring system to trigger alerts when such logs are detected. This could be based on thresholds for the number of invalid messages over a certain period.
Quarantine Mechanism: When an alert is triggered, the system should take action to quarantine the producer. This can involve updating the Kafka topic configuration (e.g., adding a __invalid_producer prefix) or using a custom Kafka Connect worker to monitor and block traffic from the problematic producer.

#### Implementation
Steps

Producer Code Integration:

JAVA
import io.confluent.kafka.serializers.KafkaAvroSerializer;
import org.apache.kafka.clients.producer.ProducerRecord;

public class SchemaValidatingProducer {
    private final KafkaAvroSerializer avroSerializer = new KafkaAvroSerializer();
    private final SchemaRegistryClient schemaRegistryClient;

    public SchemaValidatingProducer(SchemaRegistryClient schemaRegistryClient) {
        this.schemaRegistryClient = schemaRegistryClient;
    }

    public void send(String topic, String key, MyEvent event) throws Exception {
        // Validate the schema
        if (!schemaRegistryClient.validate("my-event-schema", event)) {
            throw new IllegalArgumentException("Message does not match the registered schema.");
        }

        ProducerRecord<String, byte[]> record = new ProducerRecord<>(topic, key, avroSerializer.serialize("my-event-schema", event));
        try (KafkaProducer<String, byte[]> producer = new KafkaProducer<>(producerProps)) {
            // Send the message
            producer.send(record);
        }
    }
}

Logging and Alerting:

JSON
{
  "name": "schema-validation",
  "type": "logger",
  "config": {
    "loggers": [
      {"name": "com.example.producer", "level": "INFO"}
    ]
  }
}

Quarantine Mechanism:

JAVA
import org.apache.kafka.clients.admin.KafkaAdminClient;
import org.apache.kafka.common.TopicPartition;

public class ProducerQuarantiner {
    private final KafkaAdminClient adminClient;

    public ProducerQuarantiner(KafkaAdminClient adminClient) {
        this.adminClient = adminClient;
    }

    public void quarantineProducer(String producerId, String topic) throws Exception {
        // Update the topic configuration
        NewTopic newTopic = new NewTopic(topic + "__invalid_producer", 1, (short) 1);
        adminClient.createTopics(Collections.singletonList(newTopic));

        // Block traffic from the problematic producer
        List<TopicPartition> partitions = adminClient.listConsumerGroupOffsets("my-consumer-group")
                                                        .partitionsFor(topic)
                                                        .collect(Collectors.toList());
        for (TopicPartition partition : partitions) {
            adminClient.alterPartitionAssignmentForConsumerGroup("my-consumer-group", Collections.singleton(partition));
        }
    }
}

#### Closing Summary
and Common Mistakes/Follow-Ups

This solution ensures that producers are validated in real-time, alerting us to issues as they arise. By quarantining the problematic producer, we prevent further corruption from spreading. However, it is crucial to consider potential performance impacts on the producer and consumer systems, and ensure robust logging and monitoring.

#### Common mistakes
include:

Failing to integrate schema validation at the producer level.
Not setting up effective alerting mechanisms.
Ignoring the impact on existing consumers and producers during quarantine operations.

#### Follow-ups
might involve discussing how to handle large volumes of messages, integrating with external systems for more advanced analytics, or exploring alternative quarantine strategies that minimize downtime.

**Comparison:**

```
Schema Registry Client Method	Description
validate(String schemaName, Object data)	Validates if the given data adheres to the specified schema.
```

**Code:**

```

import io.confluent.kafka.serializers.KafkaAvroSerializer; import org.apache.kafka.clients.producer.ProducerRecord;

public class SchemaValidatingProducer { private final KafkaAvroSerializer avroSerializer = new KafkaAvroSerializer(); private final SchemaRegistryClient schemaRegistryClient;

CODE
Copy
public SchemaValidatingProducer(SchemaRegistryClient schemaRegistryClient) {
    this.schemaRegistryClient = schemaRegistryClient;
}

public void send(String topic, String key, MyEvent event) throws Exception {
    if (!schemaRegistryClient.validate("my-event-schema", event)) {
        throw new IllegalArgumentException("Message does not match the registered schema.");
    }

    ProducerRecord<String, byte[]> record = new ProducerRecord<>(topic, key, avroSerializer.serialize("my-event-schema", event));
    try (KafkaProducer<String, byte[]> producer = new KafkaProducer<>(producerProps)) {
        producer.send(record);
    }
}

}

CODE
Copy

```java
import org.apache.kafka.clients.admin.KafkaAdminClient;
import org.apache.kafka.common.TopicPartition;

public class ProducerQuarantiner {
    private final KafkaAdminClient adminClient;

    public ProducerQuarantiner(KafkaAdminClient adminClient) {
        this.adminClient = adminClient;
    }

    public void quarantineProducer(String producerId, String topic) throws Exception {
        NewTopic newTopic = new NewTopic(topic + "__invalid_producer", 1, (short) 1);
        adminClient.createTopics(Collections.singletonList(newTopic));

        List<TopicPartition> partitions = adminClient.listConsumerGroupOffsets("my-consumer-group")
                                                       .partitionsFor(topic)
                                                       .collect(Collectors.toList());
        for (TopicPartition partition : partitions) {
            adminClient.alterPartitionAssignmentForConsumerGroup("my-consumer-group", Collections.singleton(partition));
        }
    }
}
```

**Example input / output:**

```text
Input:
Producer sends a message with an unregistered schema.

Output:
Log entry: Producer [producer-123] sent invalid message on topic [my-topic] at [timestamp].
Alert triggered: Invalid messages detected from producer [producer-123].

Action taken:
Quarantine the producer by renaming the topic and blocking its access.
```

**Data:**

```json
{
  "name": "schema-validation",
  "type": "logger",
  "config": {
    "loggers": [
      {"name": "com.example.producer", "level": "INFO"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Producer Sends Message
6. Is Schema Valid?
7. Kafka Producers Send Message
8. Log Invalid Message; Alert
9. Quarantine Producer?
10. Update Topic Configuration,Block Access

