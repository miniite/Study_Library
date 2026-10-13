---
type: "practice"
difficulty: "Intermediate"
---

# Describe the trade-offs between JSON Avro and Protobuf as the ingest wire format for telemetry landing in the lake.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Telemetry Lake File Layout](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Telemetry_Lake_File_Layout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering the trade-offs between JSON, Avro, and Protobuf as the ingest wire format for telemetry data in an Observability Data Lake, we must balance several factors including performance, schema evolution, storage efficiency, and complexity. JSON is simple to read and write but lacks binary serialization benefits. Avro offers better compression and more efficient schema management compared to JSON, while Protobuf excels in performance due to its compact size and statically defined schemas.

To start, let's summarize the core differences:

JSON: Human-readable format with dynamic typing; no built-in support for large-scale data handling.
Avro: Semi-static schema with better compression and more efficient parsing; supports complex data structures.
Protobuf: Static schema definition; highly optimized for performance but requires more setup.

**Comparison:**

```
Format	Human Readable	Schema Evolution	Storage Efficiency	Performance
JSON	Yes	Limited	Low	Low
Avro	Semi-static	Good	Medium	Moderate
Protobuf	Static	Limited	High	High
```

**Code:**

```
Example Protobuf message in Python

import protobuf_message as pbm

class Metric(pbm.Metric): value = 42.0

CODE
Copy
```java
// Example Avro schema and record in Java
import org.apache.avro.generic.GenericRecord;

public class Metric {
    public static GenericRecord create() {
        return new org.apache.avro.generic.GenericData.Record(new Schema.Parser().parse(Metric.SCHEMA$));
    }
}
```

**Example input / output:**

```text
# Example JSON to Avro transformation
Input (JSON):
{
  "metricName": "cpu.utilization",
  "value": 85.0,
  "timestamp": 1629734400
}

Output (Avro Record):
Metric: {
    metricName: "cpu.utilization",
    value: 85.0,
    timestamp: 1629734400
}
```

**Data:**

```
JSON Example

{ "metricName": "cpu.utilization", "value": 85.0, "timestamp": 1629734400 }

CODE
Copy

```avro
# Avro Schema
{
  "type": "record",
  "name": "Metric",
  "fields": [
    {"name": "metricName", "type": "string"},
    {"name": "value", "type": ["null", "float"]},
    {"name": "timestamp", "type": "long"}
  ]
}
```

**Flow:**

1. FLOWCHART · MERMAID
2. graph TD;
3. A[JSON] --> B1{Dynamic Schema?}
4. B1 Yes --> C[Human Readable, Limited Scalability]
5. B1 No --> D1[Avro or Protobuf]
6. D1 -- Avro --> E[Semi-static Schema, Medium Storage Efficiency, Moderate Performance]
7. D1 -- Protobuf --> F[Static Schema, High Storage Efficiency, High Performance]

