---
type: "practice"
difficulty: "Deep"
---

# How do you validate schema compatibility for JSON Schema given it has no native reader/writer resolution like Avro — what does the registry actually check?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Phased Schema Rollouts](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Phased_Schema_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

The primary goal of validating schema compatibility in a JSON Schema context involves ensuring that new schemas are backward compatible with existing data and forward compatible with future changes. The Schema Registry, in this case, acts as a central repository to track these schemas and their evolution over time.

#### Triaging the Question

The question is probing your understanding of how schema validation works for JSON Schemas without the native resolution mechanisms found in formats like Avro. Specifically, it wants you to explain the steps and processes involved in ensuring that a new schema is compatible with existing data and future schemas stored within the registry.

#### First-Principles Solution
Registry Setup: The Schema Registry stores all historical versions of schemas. Each version has a unique identifier (ID) and timestamp.
Version Tracking: When a new schema is uploaded, it's compared against previous versions to ensure backward compatibility.
Schema Validation:
Schema Semantics Check: Ensuring that the new schema adheres to semantic rules defined by JSON Schema standards.
Backward Compatibility Check: Verifying that the existing data conforms to the new schema while supporting older schemas as fallbacks.
Forward Compatibility Consideration: Ensuring the new schema can handle future changes without breaking current implementations.
Detailed Steps:
Schema Semantics Validation:
Parse both the old and new JSON Schemas using a JSON Schema validator library (e.g., jsonschema, jettison in Java).
Ensure that all properties defined in the old schema are still valid under the new schema.
Backward Compatibility:
For each piece of existing data, validate it against both the old and new schemas.
If a piece of data passes validation for the old schema but fails for the new schema, identify the specific changes required to make it compatible.
Forward Compatibility Consideration:
Designate "optional" or "new" fields in the new schema that are not strictly enforced immediately.
Allow existing systems to start using these optional fields while continuing to read and write data according to older schemas.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

In summary, the Schema Registry ensures compatibility by leveraging semantic checks, backward compatibility testing, and forward-thinking design. Common mistakes include failing to account for edge cases or overlooking subtle changes in schema semantics that can affect data integrity.

#### Follow-ups

How would you handle cyclic dependencies between schemas?
What measures do you take to minimize downtime during schema updates?

**Comparison:**

| Field     | Old Schema         | New Schema           |
|-----------|--------------------|---------------------|
| Required  | name, age          | name, age           |
| Optional  | -                  | address             |

**Code:**

```python
# Example of JSON Schema validation using jsonschema library
from jsonschema import validate, ValidationError

def validate_schema(data, schema):
    try:
        validate(instance=data, schema=schema)
    except ValidationError as e:
        return False, str(e)
    return True, "Schema validated successfully"

old_schema = {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer"}
    },
    "required": ["name", "age"]
}

new_schema = {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "integer"},
        "address": {"type": "object", "properties": {"city": {"type": "string"}, "state": {"type": "string"}}}
    },
    "required": ["name", "age"]
}

data = {"name": "John Doe", "age": 30, "address": {"city": "New York", "state": "NY"}}

is_valid_old, error_message = validate_schema(data, old_schema)
print(f"Old Schema Validation: {is_valid_old}, Error: {error_message}")

is_valid_new, error_message = validate_schema(data, new_schema)
print(f"New Schema Validation: {is_valid_new}, Error: {error_message}")
```

**Example input / output:**

```text
Old Schema Validation: True, Error: None

New Schema Validation: True, Error: None
```

**Data:**

```json
{
  "oldSchema": {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "age": {"type": "integer"}
    },
    "required": ["name", "age"]
  },

  "newSchema": {
    "$schema": "http://json-schema.org/draft-07/schema#",
    "type": "object",
    "properties": {
      "name": {"type": "string"},
      "age": {"type": "integer"},
      "address": {"type": "object", "properties": {"city": {"type": "string"}, "state": {"type": "string"}}}
    },
    "required": ["name", "age"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Upload New Schema
4. Is the new schema backwardcompatible?
5. Update Registry
6. Identify Incompatibilities
7. Adjust Existing Data to BeCompatible with New Schema
8. Notify Users of Update
9. Deploy New Schema inProduction

