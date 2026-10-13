---
type: "practice"
difficulty: "Intermediate"
---

# What are Avro and Protobuf and why might you pick one over plain JSON for Kafka messages?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Kafka vs Spark Roles](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Kafka_vs_Spark_Roles.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with big data processing, especially in a streaming context like Apache Kafka, choosing the right format for your messages is crucial. This question asks about two popular serialization formats: Avro and Protobuf. Both offer advantages over plain JSON, but the choice depends on specific use cases.

#### Triage

Question: What are Avro and Protobuf, and why might you pick one over plain JSON for Kafka messages?

What is being probed: Understanding of different data serialization formats and their trade-offs.

#### Assumptions
Familiarity with Apache Kafka and its usage in a streaming environment. Knowledge of common use cases and performance considerations.

#### First Principles Solution
Avro

Avro is a data serialization system developed by the Apache Software Foundation. It allows both flexible and strict schemas, supports efficient encoding/decoding, and provides schema evolution capabilities.

Key Features:
Schema Evolution: Ability to modify schema over time without breaking existing code.
Type Safety: Ensures type safety during runtime through reflection.
Efficient Encoding: Optimized for memory and network use, reducing the size of messages.
Protobuf

Protocol Buffers (Protobuf) is a language-agnostic, platform-neutral mechanism provided by Google. It defines a set of message types with specific fields, which can be serialized to binary format.

Key Features:
Efficient Binary Format: Optimized for performance and memory usage.
Language Support: Wide variety of supported programming languages.
Schema Evolution: Supports backward-compatible schema changes.
Plain JSON

JSON (JavaScript Object Notation) is a lightweight data interchange format. While it’s widely used, its plain text nature makes it less efficient compared to binary formats for large datasets.

Key Features:
Human Readable: Easy to read and write.
Flexible Schema: Dynamic schemas but not as strict or type-safe as Avro/Protobuf.
Why Choose One Over the Other
When to Use Avro:
Schema Evolution: If your application requires frequent changes in schema without breaking existing consumers, Avro's schema evolution capabilities are invaluable.
Type Safety and Validation: You need strong typing and validation during runtime.
Developer Experience: Avro comes with tools like avro-tools which provide convenient command-line utilities for working with schemas.
When to Use Protobuf:
Performance: For high-performance applications where every byte counts, Protobuf's binary format is more efficient.
Wide Language Support: If your application is developed in multiple languages and interoperability across teams is crucial, Protobuf’s support across a wide range of languages can be beneficial.
Backward Compatibility: You want to ensure that schema changes do not break existing consumers, especially in large-scale systems.

#### Example Scenario

Suppose you are building a real-time data pipeline using Kafka for ingesting log data from multiple services. Each service sends logs with varying structures. In this scenario:

If the schemas of these logs frequently change (e.g., adding new fields), Avro would be more suitable due to its schema evolution capabilities.
For metrics that need to be processed quickly and have fixed schemas, Protobuf could provide better performance.

#### Closing Summary

Avro and Protobuf are both powerful serialization formats with their own strengths. The choice between them depends on the specific requirements of your application, such as schema flexibility, type safety, and performance needs. While JSON is simple and human-readable, Avro and Protobuf offer more advanced features that can significantly improve data processing efficiency in large-scale systems.

#### Common Mistakes/Follow-Ups
Mistake: Assuming plain JSON always provides sufficient performance, ignoring its inefficiencies.

#### Follow-Up
Discussing specific use cases where each format would be most appropriate or exploring trade-offs between Avro and Protobuf for a given scenario.

**Code:**

```
Example of creating an Avro schema and generating Java classes

from avro.schema import parse

schema_str = """ { "name": "User", "type": "record", "fields": [ {"name": "id", "type": "int"}, {"name": "name", "type": "string"} ] } """

schema = parse(schema_str)

CODE
Copy

```java
// Generated Java classes for the above Avro schema
public class User implements org.apache.avro.specific.SpecificRecord {
  private int id;
  private String name;

  // Getters and setters...
}
```

**Data:**

```json
{
  "name": "John Doe",
  "id": 12345,
  "age": 30
}
```

