---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd evolve a Parquet/Avro schema for a table consumed by many independent jobs. Define your compatibility policy (backward/forward/full) and how a schema registry prevents a producer's change from breaking a downstream Spark reader.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Event Schema Evolution](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Event_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evolve a Parquet/Avro schema for a table consumed by many independent jobs, we need to ensure that our changes are backward and forward compatible while maintaining the integrity of data processing. This involves defining a compatibility policy and leveraging a schema registry like Apache Avro's avro-tools or Confluent's Schema Registry. The goal is to minimize disruptions in downstream processes without compromising the quality and consistency of the data.

#### Triage

The question asks us to detail how we would handle schema evolution for Parquet/Avro tables used across multiple independent jobs, ensuring compatibility with existing consumers (downstream Spark readers). We need to define our approach to backward and forward compatibility and explain how a schema registry can help manage these changes effectively.

#### First-Principles Solution
Compatibility Policy:
Backward Compatibility: Changes should not break existing data or consumers.
Forward Compatibility: New data producers can introduce new fields without breaking existing consumers.
Schema Registry Role:
Centralize schema management to ensure version control and compatibility.
Process Flow:
Schema definition and registration.
Schema versioning and tagging.
Dynamic schema fetching based on consumer requirements.
Step-by-Step Solution

Define Compatibility:

Backward Compatibility: Ensure that new columns or data types introduced do not affect existing data.

#### Example
Adding a nullable column to an existing table.
Forward Compatibility: Allow new consumers to handle additional fields without issues.

#### Example
Introducing optional fields in Avro schema.

Schema Registry Setup:

Use avro-tools or Confluent's Schema Registry for version control.
Register schemas with unique IDs and tags (e.g., v1, v2).

Data Producer Updates:

Modify the producer to use new schema versions when writing data.
PYTHON
# Example Python code using avro-tools
from avro.schema import parse

old_schema = parse(open('old_schema.avsc').read())
new_schema = parse(open('new_schema.avsc').read())

with open('data.parquet', 'rb') as f:
    data = parquet.read(f)

# Update schema and write to new file
parquet.write(data, "updated_data.parquet", writer_schema=new_schema)

#### Cons
umer Handling:

Configure consumers to use the appropriate schema version.
Use runtime schema fetching from the registry.

Schema Versioning Strategy:

Tag each version with a semantic meaning (e.g., v1, v2).
Implement a strategy for managing schema lifecycle, such as deprecation timelines.

#### Example
Schema Registry Interaction:

PYTHON
# Example Python code using Confluent's Schema Registry
from confluent_kafka.schema_registry import SchemaRegistryClient

client = SchemaRegistryClient({"url": "http://localhost:8081"})
schema_id_v1 = 123456789
new_schema_v2 = open('new_schema.avsc').read()

# Register the new schema with a version tag
response = client.register AvroSchema(new_schema_v2, schema_id=schema_id_v1)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, evolving a Parquet/Avro schema requires careful consideration of compatibility to avoid breaking existing consumers. Using a centralized schema registry ensures version control and smooth transitions. Key mistakes include:

Ignoring Backward Compatibility: Failing to handle legacy data correctly.
Overlooked Forward Compatibility: Introducing changes that break new consumers.
Inadequate Schema Versioning Strategy: Poor management of schema lifecycle leading to conflicts.

Common follow-ups might include:

How do you handle schema versioning in a distributed system?
What tools and practices would you use for testing compatibility during the transition?

**Comparison:**

```
Schema Version	Compatibility Type	Notes
v1	Backward & Forward	No breaking changes
v2	Backward & Forward	Added nullable fields
```

**Code:**

```
Example Python code using avro-tools

from avro.schema import parse

old_schema = parse(open('old_schema.avsc').read()) new_schema = parse(open('new_schema.avsc').read())

with open('data.parquet', 'rb') as f: data = parquet.read(f)

Update schema and write to new file

parquet.write(data, "updated_data.parquet", writer_schema=new_schema)

CODE
Copy

```python
# Example Python code using Confluent's Schema Registry
from confluent_kafka.schema_registry import SchemaRegistryClient

client = SchemaRegistryClient({"url": "http://localhost:8081"})
schema_id_v1 = 123456789
new_schema_v2 = open('new_schema.avsc').read()

# Register the new schema with a version tag
response = client.register_avro_schema(new_schema_v2, schema_id=schema_id_v1)
```

**Example input / output:**

```text
Input:
Parquet file containing data using the old schema.

Output:
Updated Parquet file with additional column 'age' and compatible with new downstream readers.
```

**Data:**

```json
{
  "old_schema": {
    "type": "record",
    "name": "OldRecord",
    "fields": [
      {"name": "id", "type": "int"},
      {"name": "name", "type": "string"}
    ]
  },
  "new_schema": {
    "type": "record",
    "name": "NewRecord",
    "fields": [
      {"name": "id", "type": "int"},
      {"name": "name", "type": "string"},
      {"name": "age", "type": ["null", "int"], "default": null}
    ]
  }
}
```

**Flow:**

1. Schema Registry
2. Consumer
3. Fetch Schema
4. Process Data
5. Producer
6. Write to Registry
7. Register New Schema

