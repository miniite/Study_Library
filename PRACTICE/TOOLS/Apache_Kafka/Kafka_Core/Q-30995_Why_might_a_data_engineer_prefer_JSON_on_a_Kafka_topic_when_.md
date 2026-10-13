---
type: "practice"
difficulty: "Basic"
---

# Why might a data engineer prefer JSON on a Kafka topic when starting out but move to Avro later?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Schema Formats and Registry](<../../../../TOOLS/Apache_Kafka/Kafka_Core/Schema_Formats_and_Registry.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: When first working with Apache Kafka, a data engineer might opt for JSON due to its simplicity and ease of use in various programming environments. However, as they grow more experienced, they would likely transition to using Avro, which offers better schema management, type safety, and efficient serialization.

#### Triaging the Question
The question is asking about the reasoning behind choosing between JSON and Avro for Kafka topics at different stages of a data engineer's career. We need to consider the initial ease of use with JSON compared to the advanced features offered by Avro.

Step-by-Step Solution from First Principles:

Initial Setup with JSON:

Ease of Use: JSON is lightweight and human-readable, making it easier for developers to work with, especially when starting out. It doesn't require a separate schema definition.
Flexibility: JSON can handle complex data structures without needing an external schema, which simplifies the initial development process.

Growing Experience and Need for Structured Data:

Schema Management: As a data engineer gains experience, they realize the need for better schema management to enforce data integrity and consistency.
Type Safety: Avro provides strong type safety with its schema-defined structure, which can prevent errors at runtime.
Performance: Avro offers more efficient serialization and deserialization compared to JSON, especially in high-throughput scenarios. This is crucial for large-scale Kafka implementations.

Advanced Features of Avro:

Compact Serialization: Avro supports compact binary format for messages, which can significantly reduce the size of data payloads.
Data Evolution: Avro's schema evolution features allow for changes in data structures over time without breaking existing applications, making it easier to maintain backwards compatibility.

#### Common Mistakes
and Follow-Ups:

Overlooking Schema Management: New engineers might stick with JSON due to its simplicity but overlook the importance of schema management.
Ignoring Performance Needs: As systems scale, performance bottlenecks can arise from using JSON, leading experienced data engineers to adopt Avro.

**Comparison:**

```
Encoding	Simplicity	Schema Management	Type Safety	Performance
JSON	High	None	Low	Varies
Avro	Medium	High	High	High
```

**Code:**

```python
# Example Python code for producing a Kafka message in JSON and Avro

import json
from avrogen import writer

# JSON example
json_message = {
    "name": "John Doe",
    "age": 30,
    "address": {"street": "123 Elm St", "city": "Somewhere"}
}

# Avro schema (example)
avro_schema = """
{"type":"record","name":"Person","fields":[{"name":"name","type":"string"},{"name":"age","type":"int"},{"name":"address","type":{"type":"record","name":"Address","fields":[{"name":"street","type":"string"},{"name":"city","type":"string"}]}}]}
"""

# Avro example
avro_message = writer.to_binary({'name': 'John Doe', 'age': 30, 'address': {'street': '123 Elm St', 'city': 'Somewhere'}}, avro_schema)
```

**Example input / output:**

```

Input (JSON):

JSON
Copy
{"name": "John Doe", "age": 30, "address": {"street": "123 Elm St", "city": "Somewhere"}}

Output (Avro) in Binary Format:

CODE
Copy
[154 86 0 0 ... 123 73 109 109]
```

**Data:**

```json
{
    "name": "John Doe",
    "age": 30,
    "address": {
        "street": "123 Elm St",
        "city": "Somewhere"
    }
}
```

**Flow:**

1. Initial Setup with JSON
2. Flexibility and Simplicity
3. Schema ManagementRequired as Experience Grows
4. Stay with JSON for InitialProjects
5. Migrate to Avro for BetterSchema Control
6. Continue Using JSON
7. Transition to Avro forLarge-Scale Systems

