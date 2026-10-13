---
type: "practice"
difficulty: "Deep"
---

# Producers added a required field with no default to an Avro schema and consumers started failing -- explain the compatibility rule that was violated.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Avro Schema Evolution](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Avro_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When producers add a required field without a default value in an Avro schema, it triggers strict backward compatibility rules which can lead to failure for consumers who do not have this new information. Specifically, this violates the "Avro Schema Strict Backward Compatibility" rule, which requires that any changes made to the schema (such as adding fields) must be marked with a specific tag and the field should either be optional or allow nulls.

#### Triage

The question is asking about why consumers fail when producers add a required field in an Avro schema without providing a default value. The interviewer is probing your understanding of Avro's schema evolution rules, specifically focusing on strict backward compatibility.

#### First-Principles Solution

Avro uses schema evolution to handle changes in schemas over time, but it does so with different levels of permissiveness depending on the type of change:

Adding a field: This must be marked as optional (allowing nulls) or ignore. If a required field is added without these tags, consumers will fail because they expect data to conform strictly to their schema.
Changing a field's name, type, or order: These changes need to follow specific rules to ensure backwards compatibility.

In the scenario described, producers violated the strict backward compatibility rule by adding a required field without providing a default value (null). This means consumers will encounter null values for this new field and fail because they expect non-null data in all required fields. The failure can manifest as schema validation errors or runtime exceptions during deserialization.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The core issue here is the strict requirement that any required field added to an Avro schema must either have a default value (null) or be marked with special tags allowing it to be optional. Failing to adhere to this rule can cause consumers to fail, as they expect all required fields to be present and non-null.

#### Common mistakes
include:

Forgetting to mark new required fields as optional.
Not handling null values properly when adding new required fields.
Assuming that the schema evolution rules are more permissive than they actually are.

#### Follow-up
questions might include:

How would you handle this in a real-world scenario?
What strategies could be employed to ensure compatibility while making changes?
Are there any tools or libraries that can help manage these schema changes?

**Code:**

```python
# Example of adding a required field with no default in Avro (will fail strict backward compatibility)
avro_schema = {
    "type": "record",
    "name": "ExampleRecord",
    "fields": [
        {"name": "field1", "type": "string"},
        {"name": "newField", "type": "int"}  # This is required but has no default
    ]
}
```

**Example input / output:**

```text
# Example input data that would cause failure for consumers expecting non-null values in 'newField'
{
    "field1": "example"
}

# Expected consumer behavior:
# Consumer fails with schema validation error because 'newField' is required and has no default value.
```

**Flow:**

1. Producer adds new requiredfield
2. Consumer fails due to strictbackward compatibility
3. Handling null values for newfields
4. Schema evolution strategiesfor handling changes

