---
type: "practice"
difficulty: "Deep"
---

# How would you evolve an Avro schema in a Kafka topic without breaking consumers that lag several days behind the producers?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Avro Schema Evolution](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Avro_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To evolve an Avro schema in a Kafka topic without breaking consumers that are several days behind, we need to ensure backward compatibility and avoid schema versioning conflicts. Let’s break down the approach step by step.

First, it's important to understand the current state of the system:

Current Setup: We have a Kafka topic using an Avro schema.
Issue: Consumers lagging multiple days behind could lead to schema mismatches if we make incompatible changes.

The key idea here is to introduce a mechanism that allows for smooth schema evolution while ensuring compatibility with older consumers. This can be achieved through versioning the schema and handling new fields in a way that does not break existing consumer logic.

Let’s dive into the detailed solution:

Versioned Schema Evolution:

When making changes to the Avro schema, we need to introduce a schema ID or version.
For example, if you are adding a new field newField, instead of directly updating the old schema, create a new schema with a higher version number.

Schema Registry:

Use an Apache Schema Registry (or similar) to manage and enforce versioning rules. This ensures that producers use the correct schema versions.
The Schema Registry can also be used by consumers during deserialization to handle backward compatibility gracefully.

Handling New Fields:

If you add a new field, ensure it has a default value (e.g., null for non-nullable types). This way, older consumers will ignore the new field.
Alternatively, if you need to retain older consumer behavior, consider using conditional logic within your application code to handle missing fields.

Graceful Degradation:

Implement a fallback strategy in your consumer logic to ensure that any unexpected schema changes do not cause critical failures. This can be done by logging warnings and continuing processing with the available data.

Testing:

Thoroughly test schema evolution in a staging environment before deploying to production.
Ensure that all consumers are updated to handle new schema versions.

Monitoring and Alerting:

Monitor consumer lag and schema version mismatches closely.
Set up alerts to notify when there’s significant backpressure or issues with schema handling.

#### Closing Summary
+ Common Mistakes/Follow-Ups

The key takeaway is that evolving an Avro schema in a Kafka topic requires careful planning, especially considering the potential delay between producers and consumers. Using a versioned schema approach alongside a Schema Registry ensures backward compatibility while allowing for necessary changes. Common mistakes include not handling new fields appropriately, failing to test thoroughly, or neglecting to monitor system health.

For follow-ups, it would be helpful to discuss:

Specific tools and libraries used in the process.
How to handle schema deletions or retroactive changes.

#### Trade-offs
between different versioning strategies (e.g., semantic versioning vs. incremental versioning).

**Comparison:**

```
Schema Version	Fields
1	name, favorite_number
2	name, age, favorite_num
```

**Code:**

```python
# Example of adding a new field with a default value in Avro using Python
from avro import schema, io

avro_schema = """
{
  "type": "record",
  "name": "User",
  "namespace": "example.avro",
  "fields": [
    {"name": "name", "type": "string"},
    {"name": "favorite_number", "type": ["int", "null"], "default": null}
  ]
}
"""

# Load the schema
schema_obj = schema.parse(avro_schema)
```

**Example input / output:**

```

Given an input Avro record:

JSON
Copy
{
  "name": "Jane Smith"
}

The output should be:

JSON
Copy
{
  "name": "Jane Smith",
  "favorite_number": null
}

This demonstrates how a new field is added with a default value to ensure backward compatibility.
```

**Data:**

```json
{
  "name": "John Doe",
  "favorite_number": null
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Producer
6. New Field?
7. Add Default Value
8. Use Existing Schema
9. Serialize with VersionedSchema
10. Schema Registry Validation
11. Kafka Topic
12. Consumer
13. Handle Missing Field
14. Process Data

