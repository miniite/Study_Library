---
type: "practice"
difficulty: "Intermediate"
---

# What is a schema registry and how is it used with Avro in Kafka?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Avro Schema Evolution](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Avro_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A schema registry is a tool that manages and version controls schemas for data serialization and deserialization, crucial for ensuring compatibility across different systems. In the context of using Avro with Kafka, the schema registry ensures that producers and consumers can understand and interpret messages correctly, even when the schema evolves over time.

#### Triaging the Question

The question asks you to explain what a schema registry is and how it integrates with Avro in Kafka. The interviewer might be probing your understanding of:

What schemas are and why they matter.
How Avro works for data serialization.
The role of a schema registry in managing these schemas, particularly its use cases.
Any trade-offs or considerations when implementing such a system.

#### First Principles Solution
Understanding Schemas

Schemas define the structure of data. In the context of data integration and processing, especially with distributed systems like Kafka, having consistent and well-defined schemas is critical to ensure that producers and consumers can work together seamlessly without breaking the application logic.

Avro Basics

Apache Avro is a data serialization framework used for storing and transmitting data. It includes both the schema (a description of how the data should be structured) and the binary format for efficient storage and transmission. Avro schemas are defined in JSON, making them human-readable and easy to manage.

Role of Schema Registry

A schema registry is a central repository that stores and manages these Avro schemas. It ensures that:

Schemas can be versioned: As data requirements change, new versions of the schema can be created without disrupting existing processes.
Schema evolution: Changes in the schema are controlled and tracked, ensuring compatibility across different components.
Decoupling: Producers and consumers do not need to hard-code schema details; they rely on the registry for schema discovery.
Integration with Kafka

In a Kafka ecosystem:

Schema Registration: Schemas are registered in the registry before being used by any producers or consumers.
Schema Versioning: As schemas evolve, new versions are added to the registry while old ones remain available.
Schema Discovery: Producers and consumers can retrieve the appropriate schema from the registry based on topic names.

#### Example
Workflow
A producer sends a message containing data serialized according to a specific Avro schema.
The schema is registered in the schema registry, which ensures it is compatible with existing schemas or introduces new versions if necessary.
A consumer reads messages and uses the schema stored in the registry to deserialize them correctly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

A schema registry provides a robust solution for managing data schemas in distributed systems like Kafka. It ensures that producers and consumers can work seamlessly with evolving data structures, maintaining compatibility and reducing complexity.

#### Common Mistakes

Ignoring schema versioning, leading to incompatible schema changes.
Not centralizing schema management, causing inconsistencies across different components.
Overlooking the importance of schema evolution in dynamic environments.

#### Follow-Ups

How does a schema registry handle backward and forward compatibility?
What are potential performance impacts when using a schema registry with Kafka?

**Comparison:**

```
Schema Registry Function	Description
Schema Registration	Registering Avro schemas for use in Kafka topics.
Version Control	Tracking schema versions to ensure compatibility.
Compatibility Checking	Ensuring producers and consumers are using compatible schemas.
```

**Code:**

```python
# Example Python code to interact with Confluent Schema Registry
from confluent_kafka.schema_registry import SchemaRegistryClient

def register_schema(schema_str, subject):
    client = SchemaRegistryClient({"url": "http://localhost:8081"})
    schema = client.get_latest_version(subject)
    # Register the Avro schema with the subject name
    pass
```

**Example input / output:**

```text
Input:
- Producer sends a message with data serialized according to the Avro schema for "user_profile".
Output:
- Schema is registered in the registry.
- Consumer can retrieve and use the latest version of the schema to deserialize messages correctly.

Request:
POST /subjects/user_profile/versions/latest

Response:
{
  "id": 1,
  "version": 3,
  "schema": "{\"type\":\"record\",\"name\":\"User\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"},{\"name\":\"username\",\"type\":\"string\"}]}",
  "subject": "user_profile"
}
```

**Data:**

```json
{
  "subject": "user_profile",
  "schemaId": 1,
  "version": 3,
  "schema": "{\"type\":\"record\",\"name\":\"User\",\"fields\":[{\"name\":\"id\",\"type\":\"int\"},{\"name\":\"username\",\"type\":\"string\"}]}",
  "url": "http://localhost:8081"
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. No
5. Produce Message
6. Is Schema Registered?
7. Register New Schema
8. Send Data with CorrectSchema
9. Consumer Receives Data
10. Use Registry to Get Schema
11. Deserialise and Process Data
12. Fetch Latest Schema fromRegistry

