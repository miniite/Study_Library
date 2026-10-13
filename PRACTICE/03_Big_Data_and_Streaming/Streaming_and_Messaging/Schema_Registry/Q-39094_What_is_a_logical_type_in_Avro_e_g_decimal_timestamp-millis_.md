---
type: "practice"
difficulty: "Deep"
---

# What is a logical type in Avro (e.g. decimal timestamp-millis uuid) and how do you evolve a field to a logical type without breaking older readers?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Protobuf vs Avro Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Protobuf_vs_Avro_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Avro, a logical type is an abstraction that allows us to define data in a way that's meaningful beyond its raw representation. For example, decimal, timestamp-millis, and uuid are all logical types that provide a more human-readable interpretation of the underlying primitive data. When evolving a schema to include logical types, especially when dealing with older readers (versions), we need to ensure backward compatibility.

#### Triage

The question is asking about how to introduce logical types into an Avro schema without breaking existing consumers or processors. We'll need to consider the impact on both new and old versions of code that might be reading from or writing to this data.

#### First-Principles Solution

To evolve a field to include a logical type without breaking older readers, we should follow these steps:

Define Logical Types: Understand the meaning and behavior of the logical types you want to introduce.
Version Control: Use Avro's versioning mechanism to ensure backward compatibility.
Graceful Degradation: Implement logic that allows older readers to handle new schemas gracefully.
Detailed Solution
Step 1: Define Logical Types

Logical types in Avro are designed to provide a more meaningful representation of data. For example:

decimal(10,2) can represent monetary values with two decimal places.
timestamp-millis represents timestamps as milliseconds since the Unix epoch.

Here’s an example schema that includes both primitive and logical types:

JSON
{
  "type": "record",
  "name": "ExampleRecord",
  "fields": [
    {"name": "id", "type": "long"},
    {"name": "timestamp", "type": ["null", "long"], "logicalType": "timestamp-millis"},
    {"name": "value", "type": ["null", "double"], "logicalType": "decimal(10,2)"}
  ]
}
Step 2: Version Control

Avro supports versioning by allowing schemas to be tagged with a version number. This ensures that older versions of code can continue to read the data without any issues.

When you introduce new logical types, you create a new schema version:

JSON
// Schema v1
{
  "type": "record",
  "name": "ExampleRecord_v1",
  "fields": [
    {"name": "id", "type": "long"},
    {"name": "timestamp", "type": ["null", "long"], "logicalType": "timestamp-millis"}
  ]
}

// Schema v2
{
  "type": "record",
  "name": "ExampleRecord_v2",
  "fields": [
    {"name": "id", "type": "long"},
    {"name": "timestamp", "type": ["null", "long"], "logicalType": "timestamp-millis"},
    {"name": "value", "type": ["null", "double"], "logicalType": "decimal(10,2)"}
  ]
}
Step 3: Graceful Degradation

To ensure backward compatibility, you need to handle cases where older consumers encounter new fields or logical types. This can be achieved through schema evolution techniques.

For example, in your processing logic, you might check for the presence of a field and handle its absence gracefully:

PYTHON
def process_record(record):
    id = record.get('id')
    timestamp = record.get('timestamp', None)  # Handle null timestamp
    value = record.get('value', None)         # Handle decimal conversion

    if not isinstance(timestamp, int):       # Ensure it's a valid timestamp-millis
        raise ValueError("Invalid timestamp format")

    if value is not None and isinstance(value, str):
        try:
            value = Decimal(value)
        except InvalidOperation:
            raise ValueError("Invalid decimal value")

    # Process the record as needed

#### Closing Summary

By carefully defining logical types, versioning schemas, and implementing graceful degradation techniques, you can ensure that your data schema evolution does not break older readers or processors. This approach allows for smooth transitions between different versions of your system.

#### Common Mistakes
& Follow-Ups
Failing to Handle Nulls: Not properly handling null fields in the new schema can lead to errors.
Ignoring Versioning: Failing to use version control can result in compatibility issues when upgrading readers or producers.
Overly Complex Logic: Complicated logic for handling different versions and types can introduce bugs. Simplifying where possible is key.

**Comparison:**

```
Schema Version	Fields
v1	id, timestamp
v2	id, timestamp, value
Field	Type	Logical Type
id	long	-
timestamp	long	timestamp-millis
value	double	decimal(10,2)
```

**Code:**

```python
def process_record(record):
    id = record.get('id')
    timestamp = record.get('timestamp', None)  # Handle null timestamp
    value = record.get('value', None)         # Handle decimal conversion

    if not isinstance(timestamp, int):       # Ensure it's a valid timestamp-millis
        raise ValueError("Invalid timestamp format")

    if value is not None and isinstance(value, str):
        try:
            value = Decimal(value)
        except InvalidOperation:
            raise ValueError("Invalid decimal value")

    # Process the record as needed
```

**Example input / output:**

```text
Input (Schema v1):
{
  "id": 1,
  "timestamp": null
}

Output:
Processing record for id=1 with timestamp=None

Input (Schema v2):
{
  "id": 1,
  "timestamp": 1633072800000,  // 2021-10-05T00:00:00Z
  "value": 49.99
}

Output:
Processing record for id=1 with timestamp=1633072800000 and value=Decimal('49.99')
```

**Data:**

```json
// Example data for schema v1
{
  "id": 1,
  "timestamp": null
}

// Example data for schema v2
{
  "id": 1,
  "timestamp": 1633072800000,  // 2021-10-05T00:00:00Z
  "value": 49.99
}
```

**Flow:**

1. Yes
2. No
3. Record Received
4. Is Schema V2?
5. Process Record with V2Schema
6. Handle V1 Schema Gracefully
7. Proceed with Processing
8. Convert Timestamp to Millis,Decimal to Float

