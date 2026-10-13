---
type: "practice"
difficulty: "Deep"
---

# How would you evolve the schema of a telemetry topic without breaking existing Spark consumers?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Topic and Schema Design for Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evolve the schema of a telemetry topic in Kafka without breaking existing Spark consumers, we need to carefully plan and execute a strategy that minimizes disruptions and ensures data integrity. The key is to understand how schemas are handled in Kafka topics, especially when dealing with structured logs or metrics.

#### Triage

The question essentially asks us to manage schema evolution for streaming data consumed by Spark applications. We will focus on ensuring backward compatibility, which allows new consumers to handle the updated schema while existing ones continue processing as before. This involves understanding how Apache Kafka handles schema versions and leveraging tools like Schema Registry (e.g., Confluent Schema Registry) if available.

#### Assumptions
The telemetry data is structured and stored in Kafka topics.
There are existing Spark consumers that depend on the current schema of these topics.
We aim to introduce a new, potentially incompatible schema version without breaking existing consumer workflows.

#### First-Principles Solution
Step 1: Define the Current Schema

Identify the current structure of the telemetry data being consumed by Spark. This might include fields like timestamp, deviceID, and various metrics. For example:

JSON
{
    "timestamp": long,
    "deviceID": string,
    "temperature": double,
    "humidity": double,
}
Step 2: Plan for Schema Evolution

Decide on the new schema that might include additional fields or changes to existing ones. Ensure backward compatibility by adding optional fields and using appropriate data types:

JSON
{
    "timestamp": long,
    "deviceID": string,
    "temperature": double,
    "humidity": double,
    "newField": string, // Optional field for future expansion
}
Step 3: Use Schema Registry

Leverage a schema registry to manage different versions of the schemas. This ensures that both old and new consumers can understand their respective data:

Current Consumers: Consume version 1.
New Consumers: Consume version 2.
Step 4: Gradually Rollout New Producers

Update the producers (e.g., applications or scripts) to start sending data with the updated schema. Initially, send both schemas until all consumers have been migrated:

JSON
// Version 1
{
    "timestamp": long,
    "deviceID": string,
    "temperature": double,
    "humidity": double,
}

// Version 2
{
    "timestamp": long,
    "deviceID": string,
    "temperature": double,
    "humidity": double,
    "newField": string, // Optional field for future expansion
}
Step 5: Monitor and Validate

Monitor the consumption of data by both old and new consumers. Ensure that no existing consumer logic is broken due to changes in the schema.

#### Closing Summary

By carefully managing schema versions through a schema registry, we can evolve the telemetry topic schema without breaking existing Spark consumers. This approach ensures backward compatibility and allows for gradual adoption of new features or fields.

#### Common Mistakes/Follow-Ups
Overwriting Existing Data: Be cautious about overwriting old data with new formats that might not be compatible.
Inconsistent Schema Versions: Ensure all producers are updated to send the correct schema version, otherwise, it can cause confusion and errors in consumption.

#### Cons
umer Migration: Plan for a smooth migration of consumers from old schemas to new ones, possibly using a gradual rollout strategy.

**Comparison:**

| Version | Fields                    |
|---------|---------------------------|
| 1       | timestamp, deviceID, temp, humidity    |
| 2       | timestamp, deviceID, temp, humidity, newField      |

**Code:**

```python
# Example Python producer script with schema evolution support
from confluent_kafka import Producer

producer = Producer({'bootstrap.servers': 'localhost:9092'})

def produce_data(version):
    message_value = {
        "timestamp": 1672531200,
        "deviceID": "dev-001",
        "temperature": 25.5,
        "humidity": 45.2
    }

    if version == 2:
        message_value["newField"] = "example"

    producer.produce('telemetry-topic', key='key1', value=str(message_value).encode('utf-8'))
```

**Example input / output:**

```text
# Input for version_1 consumer
Input: {"timestamp": 1672531200, "deviceID": "dev-001", "temperature": 25.5, "humidity": 45.2}

# Output from version_1 consumer
Output: Processing telemetry data for device dev-001

# Input for version_2 consumer
Input: {"timestamp": 1672531200, "deviceID": "dev-001", "temperature": 25.5, "humidity": 45.2, "newField": "example"}

# Output from version_2 consumer
Output: Processing telemetry data for device dev-001 with additional field newField = example
```

**Data:**

```json
{
    "version_1": {
        "timestamp": 1672531200,
        "deviceID": "dev-001",
        "temperature": 25.5,
        "humidity": 45.2
    },

    "version_2": {
        "timestamp": 1672531200,
        "deviceID": "dev-001",
        "temperature": 25.5,
        "humidity": 45.2,
        "newField": "example"
    }
}
```

**Flow:**

1. Produce Data
2. Version 1
3. Store in Kafka Topic
4. Consume Data (OldConsumers)
5. Process Data
6. Version 2
7. Store in Kafka Topic
8. Consume Data (NewConsumers)
9. Process Enhanced Data withnewField
10. Gradually Rollout Version 2Producers
11. Monitor and Validate SchemaEvolution

