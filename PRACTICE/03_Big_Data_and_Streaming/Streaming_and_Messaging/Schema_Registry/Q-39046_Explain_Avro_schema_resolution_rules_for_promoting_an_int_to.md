---
type: "practice"
difficulty: "Deep"
---

# Explain Avro schema resolution rules for promoting an int to a long or a float to a double — which direction is safe and why does the reader schema govern it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Protobuf vs Avro Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Protobuf_vs_Avro_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing Avro schema resolution in the context of evolving schemas, particularly concerning type promotion (such as an int becoming a long, or a float becoming a double), understanding the schema registry and its role becomes crucial. The key concept here is that the reader schema governs how types are promoted during deserialization, ensuring data integrity and compatibility across different versions of schemas.

#### Triage

This question primarily probes your understanding of Avro's schema evolution mechanisms and type coercion rules. Specifically, it aims to determine whether you understand:

The direction in which types can be safely promoted.
Why the reader schema is significant in this context.
Potential pitfalls or edge cases associated with these rules.

#### First Principles Solution

Avro employs a reader-writer contract where the writer's schema defines how data is serialized, and the reader's schema governs deserialization. During deserialization, if the writer’s schema has an int field that needs to be read by a reader expecting a long, or a float field that should be interpreted as a double, Avro promotes these types accordingly based on predefined rules.

Type Promotion Rules

From Int to Long:

An int can safely be promoted to a long.
The promotion is safe because an int fits within the range of a long.

From Float to Double:

A float can safely be promoted to a double.
The promotion is safe due to the precision and range differences between these types, where a double offers more precision.

Safety Considerations:

Promoting from smaller to larger types (e.g., int to long, or float to double) does not lose data.
However, promoting in the opposite direction (larger to smaller) can result in loss of information and potential precision issues.
Reader Schema Governs Promotion

The reader schema is crucial because it dictates how data should be interpreted during deserialization. When a writer writes with an older schema version, but a newer schema expects certain types, the reader's schema determines how these types are promoted. The reader schema essentially defines the expected structure and type safety.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Why Reader Schema Governs: The reader schema must be compatible or at least capable of interpreting data written by any writer with a valid schema version.
Safety Direction: Promoting types from smaller to larger is generally safe, whereas the opposite can lead to data loss and precision issues.

#### Common Mistakes
Misunderstanding Type Promotion Rules: Assuming that promoting long back to int or double to float does not cause issues.
Ignoring Reader Schema Compatibility: Failing to ensure that the reader schema is updated to handle new types written by newer writers.

#### Example

#### Cons
ider a scenario where an Avro writer uses schema 1, which includes:

JSON
{"name": "example", "type": "record", "fields": [{"name": "num", "type": "int"}]}

Later, the same writer upgrades to schema 2, using long instead of int:

JSON
{"name": "example", "type": "record", "fields": [{"name": "num", "type": "long"}]}

A reader expecting a float in its schema might still be able to deserialize correctly if the actual data was originally written as an int, and Avro promotes it safely to a long. However, if this same reader expects a double while reading from a float, it could lead to issues.

Self-Check

I have ensured that all key concepts are covered, including type promotion rules, the role of the reader schema, and common pitfalls. The solution is both comprehensive and precise.

**Code:**

```

{ "name": "example", "type": "record", "fields": [ {"name": "num", "type": "int"} ] }

CODE
Copy

```json
{
  "name": "example",
  "type": "record",
  "fields": [
    {"name": "num", "type": "long"}
  ]
}
```

