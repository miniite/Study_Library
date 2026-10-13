---
type: "practice"
difficulty: "Deep"
---

# Why are Avro or Protobuf often used to serialize CDC messages instead of plain JSON?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ AWS DMS vs Debezium](<../../../../02_Data_Engineering/Pipelines_and_Integration/Change_Data_Capture/AWS_DMS_vs_Debezium.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand why Avro or Protobuf are often preferred over plain JSON for serializing Change Data Capture (CDC) messages, we need to consider several factors that impact performance, data integrity, and ease of use in distributed systems. Avro and Protobuf provide more efficient serialization formats with built-in schema support, whereas JSON is a human-readable format but can be less efficient when dealing with large volumes of data.

Firstly, let's restate the core question: Why are Avro or Protobuf used instead of plain JSON for CDC messages? The interviewer might be probing into the performance benefits, space efficiency, and schema handling capabilities provided by these binary formats. We will also consider assumptions such as the nature of CDC data and the environments in which CDC is implemented.

Now, let's break down the solution from first principles:

Performance: Avro and Protobuf are designed to be more compact than JSON. They use less space for the same amount of data, which reduces network overhead and storage requirements. For example, a simple struct with multiple fields will serialize into fewer bytes in binary formats compared to the equivalent JSON.

Schema Handling: Both Avro and Protobuf provide built-in schema mechanisms that allow you to define the structure of your data explicitly. This means that the receiver can validate the incoming message against the expected schema, ensuring data integrity even if a field is missing or an unexpected type is used. In contrast, JSON relies on the client to ensure that all necessary fields are present and correctly typed.

Versioning: CDC data often needs to be compatible across different versions of the system. Protobuf supports versioned schemas, allowing you to evolve the schema over time without breaking existing consumers. Avro also supports schema evolution through its concept of "forward" and "backward" compatibility.

Compression and Encoding: Both Avro and Protobuf can integrate well with compression algorithms like Snappy or Gzip, which can further reduce the size of serialized data during transmission. JSON does not natively support such integration as easily, requiring additional steps to compress and decompress the data.

Error Handling: The binary formats are more robust in terms of error handling. They provide better support for partial messages and can handle cases where data is corrupted or incomplete, whereas JSON might require additional logic to handle these scenarios.

Readability and Maintenance: While Avro and Protobuf are not as human-readable as JSON, they offer significant benefits in terms of performance and reliability. For a data engineer dealing with large volumes of CDC data, the trade-off between readability and efficiency is often justified.

In summary, Avro or Protobuf are preferred for CDC messages because they provide better performance, support schema validation and versioning, integrate well with compression, handle errors more robustly, and require less maintenance in complex distributed systems. The choice between Avro and Protobuf might depend on specific requirements such as the need for strict typing, ease of use, or compatibility with existing tools.

#### Common mistakes
to avoid include assuming that JSON is always sufficient due to its simplicity, ignoring the importance of schema validation, and underestimating the impact of data size and performance in large-scale systems.

**Code:**

```python
# Example Protobuf definition for a simple user message
syntax = "proto3";

message User {
  int32 id = 1;
  string name = 2;
  repeated string emails = 3;
}
Avro schema for the same user message

{ "type": "record", "name": "User", "fields": [ {"name": "id", "type": "int"}, {"name": "name", "type": "string"}, {"name": "emails", "type": ["null", {"type": "array", "items": "string"}], "default": null} ] }
```

**Example input / output:**

```

No explicit input-output examples are needed for this explanation, but here is a sample scenario to illustrate:

JSON Example

JSON
Copy
{
  "id": 1,
  "name": "Alice",
  "emails": ["alice@example.com"]
}

Avro Example

AVRO
Copy
{"id": 1, "name": "Alice", "emails": ["alice@example.com"]}

Protobuf Example

PROTOBUF
Copy
message User {
  int32 id = 1;
  string name = 2;
  repeated string emails = 3;
}

User user = User.newBuilder()
               .setId(1)
               .setName("Alice")
               .addEmails("alice@example.com")
               .build();
```

**Flow:**

1. Yes
2. No
3. Serialization
4. Is Data Large?
5. Choose Avro/Protobuf
6. Use JSON for Simplicity
7. Avro Provides SchemaValidation and Versioning
8. Integrates Well withCompression
9. Robust Error Handling
10. Simple, Human-Readable

