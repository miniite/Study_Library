---
type: "practice"
difficulty: "Deep"
---

# Explain how a compatibility check works in a schema registry when a producer registers a new schema version.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Data Contract Practice](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Contracts/Data_Contract_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a producer registers a new schema version with a schema registry, the compatibility check ensures that this new version is backward and forward compatible with existing data consumers and older versions of schemas. Compatibility checks are crucial in systems where multiple services consume from or produce to shared data stores, ensuring seamless integration and preventing potential runtime errors.

#### Triaging the Question

The question asks about how a compatibility check works when a producer registers a new schema version in a schema registry. The core aspects being probed include:

Understanding of Schema Registry Concepts: Knowledge of what a schema registry is and its role.
Schema Versioning and Compatibility: Familiarity with different types of compatibility (backward, forward, full) and their implications.

#### Implementation
Details: How the actual check mechanism works.

#### Assumptions
The schema registry supports multiple versions of schemas for a given topic or resource.

#### Cons
umers expect a certain level of backward/forward compatibility from new schemas.
There are predefined rules for determining whether new schemas are compatible with existing ones.

#### First-Principles Solution
1. Schema Definitions and Registry

A schema registry maintains definitions of data schemas used in the system. These definitions include:

Schema ID: A unique identifier for a specific schema version.
Schema Data: The actual schema definition, often in JSON or Avro format.

When a producer wants to register a new schema, it sends this schema along with metadata such as compatibility level (backward, forward, full) and optional documentation.

2. Compatibility Levels

The schema registry supports different levels of compatibility:

Backward Compatible: New schema should be able to replace old schemas without breaking existing consumers.
Forward Compatible: Old consumers should be able to handle new schema versions correctly.
Full Compatibility: The most stringent level, requiring full compatibility with all previous and future versions.
3. Versioning Strategy

Producers register a new schema version by incrementing a version number. The registry stores these versions and uses them for determining compatibility:

Each version has a unique ID, allowing multiple versions of the same schema.

#### Cons
umers can specify which version they are using and expect certain levels of compatibility.
4. Registration Flow
Schema Validation: The producer’s schema is validated against predefined rules (e.g., Avro schema validation).
Compatibility Check:
The registry checks if the new schema is compatible with existing versions according to the specified compatibility level.
For backward compatibility, it ensures that old data can be converted to the new format without errors.
For forward compatibility, it ensures that new data produced by this version can be consumed by older consumers.
Metadata Update: If the check passes, the registry updates its metadata with the new schema and version information.
5. Implementation Details
Backward Compatibility Check:
Convert old data to the new schema format using predefined logic (e.g., mapping fields).
Ensure no loss of critical data during conversion.
Forward Compatibility Check:
Validate that the new schema does not introduce fields or changes that would break existing consumers.
Optionally, provide default values for missing fields in old data.

#### Closing Summary

In summary, compatibility checks in a schema registry ensure smooth evolution of schemas over time by verifying that new versions are compatible with both past and future usage. This is achieved through strict validation rules and detailed versioning strategies, ensuring robustness and reliability in distributed systems.

#### Common Mistakes/Follow-Ups
Ignoring Backward Compatibility: Overemphasis on forward compatibility can break existing consumers.
Inadequate Forward Compatibility: Failing to account for potential issues with new data formats might lead to runtime errors.
Schema Versioning Strategy: Poorly designed versioning strategies can complicate schema management and migration.

**Comparison:**

| Schema Version | Fields       |
|----------------|--------------|
| Old            | [field1, field2]  |
| New            | [field1, field2, field3]  |

**Code:**

```python
# Example of a simple compatibility check function in Python

def is_backward_compatible(old_schema, new_schema):
    # Convert old data to the new schema format using predefined logic
    try:
        old_data_converted = convert_old_data_to_new_format(old_data)
        return True  # Assuming no errors during conversion indicates backward compatibility
    except ConversionError as e:
        return False

def is_forward_compatible(new_schema, old_data):
    # Validate that the new schema does not introduce fields or changes that would break existing consumers
    try:
        validate_new_data_against_old_schema(new_data)
        return True  # Assuming no validation errors indicates forward compatibility
    except ValidationFailed as e:
        return False
```

**Example input / output:**

```text
Input:
- Old Schema (JSON): {"fields": [{"name": "field1", "type": "string"}, {"name": "field2", "type": "int"}]}
- New Schema (JSON): {"fields": [{"name": "field1", "type": "string"}, {"name": "field2", "type": "int"}, {"name": "field3", "type": "double"}]}

Output:
- Backward Compatibility Check: True
- Forward Compatibility Check: False  # Assuming field3 is not handled by old consumers

Dry Run:
1. Convert old data to new schema format.
2. Validate new data against old schema.
```

**Data:**

```json
{
  "oldSchema": {
    "type": "record",
    "name": "OldRecord",
    "fields": [
      {"name": "field1", "type": "string"},
      {"name": "field2", "type": "int"}
    ]
  },
  "newSchema": {
    "type": "record",
    "name": "NewRecord",
    "fields": [
      {"name": "field1", "type": "string"},
      {"name": "field2", "type": "int"},
      {"name": "field3", "type": "double"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Schema Registration
4. Is Compatibility Level Set?
5. Validate Schema
6. Check Backward Compatibility
7. Check Forward Compatibility
8. Register New Version inRegistry
9. Use Default Schema

