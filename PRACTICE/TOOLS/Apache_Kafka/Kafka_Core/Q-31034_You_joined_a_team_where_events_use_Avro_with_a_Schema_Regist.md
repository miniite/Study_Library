---
type: "practice"
difficulty: "Intermediate"
---

# You joined a team where events use Avro with a Schema Registry. How does that help keep the warehouse loads reliable over time?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Schema Formats and Registry](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Schema_Formats_and_Registry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Using Kafka with an Avro schema registry ensures data consistency and reliability by defining, validating, and evolving schemas. This helps maintain a single source of truth in your data warehouse, reducing issues like schema drift.

#### Triaging the Question

The question is asking about the benefits of using Kafka alongside Avro and a Schema Registry for maintaining reliable data loads in a data warehouse. The core idea is to ensure that data remains consistent as it moves through various stages from source to storage.

#### Assumptions

We are dealing with event-driven systems where real-time or near-real-time data processing is critical.
The team uses Kafka for its messaging infrastructure, and Avro is chosen due to its schema flexibility.
A Schema Registry is in place to manage and version schemas.

#### First Principles Solution
Step 1: Understanding Avro and Schema Registry

Avro is a data serialization framework that provides support for complex datatypes. It allows for self-describing data structures, making it easier to integrate with various systems. The Schema Registry acts as a central authority for managing these schemas.

Step 2: Ensuring Data Consistency

When events are produced in Kafka, they include Avro-encoded payloads. These payloads reference schema IDs stored in the Schema Registry. By doing this, you ensure that:

The data format is known and consistent across producers.

#### Cons
umers can validate the incoming data against the correct schema version.
Step 3: Handling Schema Evolution

In a dynamic environment, schemas need to evolve over time as new fields are added or existing ones modified. A Schema Registry allows for versioning of schemas. This means:

New versions of a schema can be deployed without breaking existing consumers.

#### Cons
umers can switch to the latest schema version if needed.
Step 4: Preventing Schema Drift

Schema drift occurs when data is stored in different formats over time, leading to inconsistencies and potential errors. By using a Schema Registry:

You maintain a single source of truth for schemas.
You can enforce rules about which versions of schemas should be used at any point in time.
Step 5: Enabling Efficient Warehouse Loads

With consistent and well-defined schemas, loading data into the warehouse becomes more reliable. This is because:

The schema ensures that all incoming data conforms to expected formats.
Schema validation at the Kafka producer level can catch errors early, reducing latency issues during data ingestion.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, using Avro with a Schema Registry in Kafka helps maintain reliable and consistent data loads by ensuring:

Data consistency across producers and consumers.
Efficient schema evolution without breaking existing processes.
Reliable data ingestion into the warehouse.

#### Common Mistakes

Ignoring Schema Evolution: Not planning for how schemas will change over time can lead to significant issues when new versions are deployed.
Inadequate Validation: Failing to validate incoming data against the correct schema version at the consumer end can result in silent failures and incorrect processing.

#### Follow-Up
Questions:

How do you handle backward compatibility when evolving a schema?
What steps would you take if you needed to roll back to an older schema version?

**Comparison:**

| Producer | Consumer |
| --- | --- |
| Produce Avro-encoded messages with schema IDs. | Validate incoming data against the correct schema version from the registry. |
| Use CachedSchemaRegistryClient for efficient schema registration and lookup. | Ensure the consumer uses the latest or appropriate schema version. |
| Register schemas in the Schema Registry before producing any data. | Handle backward compatibility if evolving a schema. |

**Code:**

```python
# Example of producing data with Avro and referencing the Schema Registry in Kafka

from kafka import KafkaProducer
from confluent_kafka.avro import AvroProducer, CachedSchemaRegistryClient

schema_registry_client = CachedSchemaRegistryClient("http://localhost:8081", cache_max_size=100)
avro_producer = AvroProducer(config={
    'bootstrap.servers': 'localhost:9092',
    'schema.registry.url': 'http://localhost:8081'
})

record_key_schema_str = """
{
  "type": "string"
}
"""

record_value_schema_str = """
{
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "id", "type": "int"},
    {"name": "name", "type": "string"}
  ]
}
"""

# Register schemas in the Schema Registry
schema_id_key = schema_registry_client.register("user-key", record_key_schema_str)
schema_id_value = schema_registry_client.register("user-value", record_value_schema_str)

avro_producer.produce(
    topic="test-topic",
    value={"id": 1, "name": "John Doe"},
    key="key-01",
    value_schema=schema_id_value,
    key_schema=schema_id_key
)
```

**Example input / output:**

```text
# Example of a producer producing data with Avro encoding

Producer Output:
{
  "key": "key-01",
  "value": {
    "id": 1,
    "name": "John Doe"
  },
  "schema_id_value": 23456789,
  "schema_id_key": 12345678
}

# Example of a consumer validating data against the schema

Consumer Input:
{
  "key": "key-01",
  "value": {
    "id": 1,
    "name": "John Doe"
  },
  "schema_id_value": 23456789,
  "schema_id_key": 12345678
}

Consumer Output:
Value: {"id": 1, "name": "John Doe"}
Schema ID: 23456789 (Valid)
Key Schema ID: 12345678 (Valid)
```

**Data:**

```json
{
  "user-key": {
    "type": "string"
  },
  "user-value": {
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

1. Yes
2. Yes
3. No
4. No
5. Producer
6. Is schema registered?
7. Encode message with Avroand reference schema ID
8. Send message to Kafka topic
9. Consumer
10. Is the correct schema versionavailable?
11. Validate incoming data againstschema
12. Process valid data
13. Fail or request new version
14. Register schema in SchemaRegistry

