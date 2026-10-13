---
type: "practice"
difficulty: "Expert"
---

# Compare how Protobuf and Avro treat a field that's present in the writer but absent in the reader — which drops it silently and which errors?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Protobuf vs Avro Evolution](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Protobuf_vs_Avro_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When dealing with data serialization formats, understanding how they handle missing fields is crucial. This problem involves comparing Protobuf and Avro to determine their behavior when a field exists in one side but not the other. Specifically, we'll look at whether these formats drop or error out on such discrepancies.

#### Triage

The core of this question revolves around how Protobuf and Avro handle optional fields during serialization and deserialization. The key is understanding that Protobuf silently drops absent fields, while Avro can raise an error if it encounters a missing required field in the schema.

#### First-Principles Solution
Protobuf Behavior

Protobuf (Protocol Buffers) uses a strongly-typed approach where each message has a fixed set of fields. If a writer includes a field that is not present in the reader's schema, Protobuf silently omits this field during deserialization. This means that if a field is optional and missing, it does not impact the overall data structure.

Avro Behavior

Avro differs by using dynamic schemas at runtime. When serializing to a union of types (e.g., null | record), if a field is present in the writer's schema but absent from the reader's, this will result in an error during deserialization. This is because Avro requires all fields specified in the schema to be present and must match their expected types.

#### Example
A Field Present in Writer but Absent in Reader

Let’s consider a concrete example where we have two schemas:

Writer Schema (Avro):

JSON
{
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "id", "type": "int"},
    {"name": "name", "type": "string"}
  ]
}

Reader Schema (Avro):

JSON
{
  "type": "record",
  "name": "User",
  "fields": [
    {"name": "id", "type": "int"},
    {"name": "email", "type": "string"}
  ]
}

Data to be written:

JSON
{
  "id": 1,
  "name": "Alice"
}

When this data is deserialized using the reader schema, it will result in an error because email is missing.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary:

Protobuf silently drops fields that do not match the receiver's schema.
Avro requires all fields to be present according to the schema; if a field is missing, it will raise an error.

Mistakes and follow-ups often involve confusion around schema validation. It’s important to note that Protobuf does not perform schema validation during deserialization, whereas Avro enforces strict schema compliance.

Additional questions might include:

How can we handle this in practical scenarios?
What are the implications of each approach on data consistency and error handling?

**Code:**

```
PROTOBUF
Copy
// Example Protobuf message (user.proto)
syntax = "proto3";
package example;

message User {
  int32 id = 1;
  string name = 2; // This field is present in writer but can be omitted by reader
}
```

**Flow:**

1. Contains id, name
2. Reads reader schema
3. Errors if email is missing
4. Writer Schema (Avro)
5. Reader Schema (Avro)
6. Data Written
7. Deserialization Error

