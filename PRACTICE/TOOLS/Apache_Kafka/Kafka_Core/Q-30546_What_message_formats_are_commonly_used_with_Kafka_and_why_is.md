---
type: "practice"
difficulty: "Intermediate"
---

# What message formats are commonly used with Kafka and why is Avro popular?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ When Not to Use Kafka](<../../../../TOOLS/Apache_Kafka/Kafka_Core/When_Not_to_Use_Kafka/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what message formats are commonly used with Apache Kafka, we need to first grasp the key concepts of messaging systems like Kafka. Typically, messages in a streaming platform like Kafka can be serialized into various formats depending on the requirements of the application. Among these, Avro is particularly popular due to its benefits such as schema evolution and efficient data serialization.

#### Triaging the Question

The question asks about common message formats used with Kafka and why Avro is favored over other options. The interviewer might be probing your understanding of the different serialization formats available in Kafka, their pros and cons, and how they are chosen based on specific use cases. We need to assume that you have a basic understanding of Kafka's architecture and its primary use cases.

#### Solution
From First Principles

Kafka messages can be serialized using various formats such as JSON, Protobuf (Protocol Buffers), Avro, MessagePack, etc. However, among these, Avro stands out for several reasons:

Schema Evolution: One of the most significant advantages of Avro is its schema evolution feature. It allows you to modify schemas over time without breaking existing consumers or producers. This is crucial in dynamic environments where data structures can change frequently.
Data Integrity and Validation: Avro provides strong typing and validation mechanisms, ensuring that the data conforms to the expected structure before it is stored or processed. This helps in maintaining data integrity and consistency across the system.
Compact Serialization: Avro uses a compact binary format for serialization, which reduces the size of messages significantly compared to plain text formats like JSON.
Why Avro?

To illustrate why Avro is popular, let's consider an example scenario where we need to process large volumes of data in real-time. Suppose you are building a recommendation engine that processes user interaction logs from various sources. These logs might change over time as new features are added or removed. With Avro, you can define a schema for the initial set of fields and evolve it as needed without disrupting ongoing operations.

Here's a simple example of how this would look in practice:

JSON
{
  "namespace": "com.example.recommendation",
  "name": "UserInteraction",
  "type": "record",
  "fields": [
    {"name": "userId", "type": "string"},
    {"name": "itemId", "type": "string"},
    {"name": "timestamp", "type": "long"}
  ]
}

When this schema changes to include more fields, you can update the schema without breaking existing consumers:

JSON
{
  "namespace": "com.example.recommendation",
  "name": "UserInteraction",
  "type": "record",
  "fields": [
    {"name": "userId", "type": "string"},
    {"name": "itemId", "type": "string"},
    {"name": "timestamp", "type": "long"},
    {"name": "eventType", "type": "enum", "symbols": ["VIEW", "BUY"]}
  ]
}

This flexibility is a major reason why Avro is preferred in many production systems.

#### Closing Summary
and Common Mistakes

In summary, Kafka supports multiple serialization formats, but Avro is widely used due to its robust schema evolution capabilities, strong typing, and efficient binary serialization. However, it's important to note that the choice of format depends on specific use cases and requirements such as performance needs, ease of development, and data integrity concerns.

#### Common mistakes
in answering this question include:

Overlooking Avro’s schema evolution feature.
Failing to mention key advantages like strong typing and compact serialization.
Not providing real-world examples or use cases where Avro would be particularly beneficial.

**Code:**

```

{ "namespace": "com.example.recommendation", "name": "UserInteraction", "type": "record", "fields": [ {"name": "userId", "type": "string"}, {"name": "itemId", "type": "string"}, {"name": "timestamp", "type": "long"} ] }

CODE
Copy

```json
{
  "namespace": "com.example.recommendation",
  "name": "UserInteraction",
  "type": "record",
  "fields": [
    {"name": "userId", "type": "string"},
    {"name": "itemId", "type": "string"},
    {"name": "timestamp", "type": "long"},
    {"name": "eventType", "type": "enum", "symbols": ["VIEW", "BUY"]}
  ]
}
```

