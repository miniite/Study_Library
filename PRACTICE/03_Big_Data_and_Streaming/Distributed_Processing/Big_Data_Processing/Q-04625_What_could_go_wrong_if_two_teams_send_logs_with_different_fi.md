---
type: "practice"
difficulty: "Basic"
---

# What could go wrong if two teams send logs with different field names into the same Kafka topic?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Topic and Schema Design for Pipelines](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Topic_and_Schema_Design_for_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When two teams send logs with different field names into the same Kafka topic, it can lead to several issues ranging from data ambiguity and processing failures to system-level problems. In this brief overview, we will delve into the core problem, identify potential pitfalls, and outline a step-by-step solution from first principles.

TL;DR: Mismatched field names in Kafka logs can cause confusion during data ingestion, processing errors in Spark jobs, and difficulty in maintaining coherent analytics pipelines. Proper schema management is crucial to avoid these issues.

#### Triaging the Question

The question probes your understanding of how data uniformity affects big data processing systems like those built with Apache Kafka and Apache Spark. Specifically, it examines your knowledge of potential mismatches that can arise when different teams produce logs using non-standardized field names, which could lead to significant operational challenges.

#### First-Principles Solution

Data Ingestion Ambiguity: When two or more teams send logs with differing field names into the same Kafka topic, it introduces ambiguity. For example, if one team uses "requestID" and another uses "sessionID", a common Spark job might not correctly map these fields.

Processing Failures: Mismatched field names can cause processing failures in Spark jobs due to incorrect schema inference or missing data. This is especially problematic when dealing with dynamic schemas like those found in Kafka, where the structure of messages may change frequently.

Analytics Challenges: Inconsistent field naming across teams makes it difficult to create unified analytics pipelines. It complicates aggregations and joins, leading to potential data misalignment issues that could skew results or lead to incorrect insights.

Maintenance Issues: Over time, maintaining a consistent schema becomes increasingly challenging as more teams join the project. This can result in spaghetti code where multiple schemas are hard-coded into different parts of the system, making it difficult to scale and update consistently.

#### Closing Summary
and Common Mistakes

To mitigate these issues, it is essential to establish standardized data models for Kafka topics across all contributing teams. This involves:

Defining a Common Schema: Agree on a common schema that both teams can adhere to.
Using Avro or Protobuf: These formats provide a robust way to define and enforce schemas, ensuring consistency.
Automated Validation Tools: Implement tools that validate incoming data against the defined schema before processing.

#### Common mistakes
include:

Failing to establish initial agreements on data models.
Ignoring schema validation in the processing pipeline.
Not regularly reviewing and updating schemas as new teams join or requirements change.

#### Example Scenario

#### Cons
ider two teams, Team A and Team B, sending logs into the same Kafka topic named userActivity. Team A uses fields like requestID, timestamp, and userID, while Team B uses sessionID, ts, and uid. If a Spark job attempts to process this data without proper schema enforcement, it might fail due to ambiguous mappings or missing values.

Closing Statement

Proper schema management is crucial for ensuring seamless big data processing. By establishing clear agreements and leveraging robust tools, we can avoid the pitfalls of mismatched field names in Kafka logs and maintain a coherent analytics pipeline.

**Comparison:**

```
Field	Team A	Team B
requestID	√	X
timestamp	√	X
userID	√	X
sessionID	X	√
ts	X	√
uid	X	√
```

**Code:**

```python
# Example Python code snippet to demonstrate schema validation using Avro
from avro.datafile import DataFileReader
from avro.io import DatumReader

class SchemaValidator:
    def __init__(self, schema_file):
        with open(schema_file, 'rb') as f:
            self.schema = DatumReader().read(f)

    def validate(self, data):
        reader = DataFileReader(data, self.schema)
        for record in reader:
            # Process each valid record
            pass

validator = SchemaValidator("path/to/schema.avsc")
validator.validate("/path/to/data.avro")
```

**Example input / output:**

```text
Input:
- Team A log: {"requestID": "12345", "timestamp": "2023-10-01T12:00:00Z", "userID": "user001"}
- Team B log: {"sessionID": "67890", "ts": "2023-10-01T12:00:00Z", "uid": "user001"}

Output:
- Validated and processed correctly by a schema-enforced Spark job
```

**Data:**

```json
{
  "requestID": "12345",
  "timestamp": "2023-10-01T12:00:00Z",
  "userID": "user001"
}
```

**Flow:**

1. Team A
2. Send logs to Kafka topic
3. Team B
4. Send logs to Kafka topic
5. Kafka Topic `userActivity`
6. Validate schema using Avro
7. Fully Validated Data
8. Spark Job for Processing

