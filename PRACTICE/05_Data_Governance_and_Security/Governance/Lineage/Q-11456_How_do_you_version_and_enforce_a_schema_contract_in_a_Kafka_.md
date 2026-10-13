---
type: "practice"
difficulty: "Intermediate"
---

# How do you version and enforce a schema contract in a Kafka pipeline? What role does the schema registry play in governance?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Glossary Metadata and Contracts](<../../../../05_Data_Governance_and_Security/Governance/Lineage/Glossary_Metadata_and_Contracts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To version and enforce a schema contract in a Kafka pipeline, especially at an intermediate experience level, we need to ensure that our data is consistently formatted across different stages of processing while maintaining a clear lineage of changes. This involves several key steps:

First, let's understand the problem more deeply. The core challenge here is ensuring that all components in your Kafka-based ETL (Extract, Transform, Load) pipeline are using the correct schema version at any given time. Additionally, we want to maintain a history of schema changes and their impacts on data lineage.

The schema registry plays a crucial role in this process by providing a central repository for managing these schemas. It ensures that the right schema is used for producing or consuming messages in Kafka topics, thus enforcing consistency across your pipeline components.

#### Triage
What the question is asking: How to manage and enforce consistent schema versions within a Kafka-based ETL pipeline.
Key concepts: Schema versioning, data lineage, and the role of the schema registry.

#### Assumptions
We are working with an existing Kafka setup.
The use of Avro or JSON as our primary schema format.
A distributed system where multiple producers and consumers might interact.

#### Solution

To manage schemas effectively in a Kafka pipeline, we can follow these steps:

Schema Registry Setup:

Introduction: Apache Kafka's Schema Registry (part of the Confluent Platform) is a REST API that stores and manages metadata about your schemas. It supports both Avro and JSON schema formats.
Usage: Producers and consumers will use this registry to fetch the latest schema for a topic.

Schema Versioning:

Introduction: Each time you make changes to a schema, it should be versioned (e.g., schema-1), ensuring that different versions can coexist without conflicts.
Process: When updating a schema, increment its version number and update the registry with the new version. Consumers will use the schema version they are configured for.

Schema Registration:

Registration Process: Before publishing data to Kafka, register the schema in the registry. This ensures that all producers and consumers are aligned on the schema definition.

#### Example
If you have a customer schema with fields like id, name, email, and now need to add phone_number, create version 2 of the schema.

Schema Validation:

Introduction: Ensure that all data published to Kafka topics conforms to the registered schema.

#### Implementation
Use a schema validation library (like Confluent's Schema Registry) that can be integrated into your pipeline’s ETL processes to validate incoming and outgoing messages against the registered schemas.

Data Lineage:

Introduction: Data lineage is crucial for understanding how data flows through different stages of processing.

#### Implementation
Maintain a record of schema versions used in each topic, along with any transformations applied during ETL processes. This helps in tracing data back to its original source and understanding any changes made.

#### Closing Summary

In summary, the schema registry is essential for managing schemas in a Kafka-based pipeline by providing version control, validation, and ensuring consistency across producers and consumers. By integrating these practices into your ETL processes, you can maintain robust data governance and lineage, which are critical for compliance and operational efficiency.

#### Common Mistakes/Follow-Ups
Mistake: Not registering schemas in the registry before using them, leading to untracked schema changes.

#### Follow-Up
Discussing how to handle backward compatibility with different consumers or producers.
Mistake: Ignoring data lineage by not maintaining a clear history of schema versions and transformations.

#### Follow-Up
Explaining the importance of data lineage in traceability and debugging.

**Comparison:**

| Topic    | Partitions | Replication Factor | Schema Version |
|----------|------------|--------------------|----------------|
| customer_updates | 3          | 1                  | 2              |

**Code:**

```python
# Example of registering a schema with Confluent Schema Registry using Python
from confluent_kafka.schema_registry import SchemaRegistryClient

# Initialize the client
schema_client = SchemaRegistryClient({'url': 'http://localhost:8081'})

# Define your Avro schema (as JSON)
avro_schema_str = """
{
  "type": "record",
  "name": "Customer",
  "fields": [
    {"name": "id", "type": "int"},
    {"name": "name", "type": "string"},
    {"name": "email", "type": "string"}
  ]
}
"""

# Register the schema
schema_id = schema_client.register_schema('Customer', avro_schema_str)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Produce a message with `customer` schema version 2 to `customer_updates` topic. | Validate the message against version 2 of the `Customer` schema in the registry. If valid, accept and store; otherwise, reject or log an error. |

**Data:**

```json
{
  "topic": "customer_updates",
  "partitions": 3,
  "replication_factor": 1,
  "schema_version": 2,  # Version of the registered schema
  "schema_registry_url": "http://localhost:8081"
}
```

**Flow:**

1. Invalid
2. Valid
3. Produce Message
4. Schema Valid?
5. Reject Message
6. Store Message in Kafka
7. Fetch Latest Schema fromRegistry
8. Validate Against Schema

