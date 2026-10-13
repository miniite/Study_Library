---
type: "practice"
difficulty: "Deep"
---

# Walk through the conversion boundary in a pipeline that ingests Avro from Kafka and must land analytics-ready Parquet -- where do type mismatches usually bite?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Avro Schema Evolution](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Avro_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When converting data from Avro to Parquet, one of the most common pitfalls is encountering type mismatches between the source (Avro) and target (Parquet). This can lead to data loss, corruption, or misleading results in downstream analytics pipelines. To address this effectively, we need to understand the fundamental differences between these formats and the potential issues that arise during conversion.

To walk through a typical scenario where Avro data is ingested from Kafka and converted to Parquet, let’s break down the process:

Avro Schema: In Avro, schemas can be flexible, allowing for schema evolution over time. This means you might have different versions of the same schema in your dataset.
Parquet Schemas: Parquet has a more rigid schema structure where each column must have a specific type defined at the time of file creation.

The key boundary is how these two systems handle data types and schemas during ingestion, transformation, and storage.

#### Triage
Question Core: The primary question here is to identify common issues that arise due to type mismatches between Avro and Parquet.
Interviewer Probing: Understanding the schema evolution strategy in Avro, how type conversions are handled, and potential pitfalls in migrating from a flexible format like Avro to a more rigid format like Parquet.

#### First Principles Solution

Avro Schema Evolution:

Avro allows for schema evolution by maintaining compatibility rules that ensure backward and forward compatibility.
For example, adding new fields or changing the type of existing fields can lead to issues if not managed correctly during conversion.

Parquet Column Types:

Parquet columns are strongly typed, meaning each column has a fixed data type which is defined at schema creation time.
If you have an Avro schema with evolving types that don't align well with the Parquet schema, it can cause mismatches during conversion.

Common Type Mismatches:

Nullable Types: In Avro, fields can be nullable (null), whereas in Parquet, columns are not nullable by default.
Precision Differences: Decimal types in Avro might have different precision levels when converted to Parquet.
Complex Types: Avro supports complex nested structures that may not map directly to a simple columnar format like Parquet.

Handling Mismatches:

Schema Validation: Use schema validation tools during the ETL process to catch mismatches early.
Data Transformation: Apply data transformations to convert Avro schemas into compatible Parquet schemas. This might involve creating intermediate steps or using tools that can handle schema evolutions gracefully.

#### Closing Summary
and Common Mistakes
Mistakes:
Failing to validate schemas during ETL processes, leading to silent failures.
Not considering the implications of type mismatches on downstream analytics.
Overlooking schema evolution rules in Avro, causing unexpected behavior in Parquet files.

#### Example Scenario

#### Cons
ider an Avro schema with a nullable integer field:

JSON
{
  "name": "data",
  "type": ["null", "int"]
}

When converting to Parquet, this must be handled as a non-nullable integer. If the original data contains null values, these need to be appropriately managed.

Common Follow-Ups
What tools or libraries do you use for schema validation?
How do you handle complex Avro schemas during conversion?
What strategies do you employ to ensure backward compatibility in Parquet files?

**Comparison:**

```
Avro Schema	Parquet Schema
nullable int	non-nullable int
```

**Code:**

```python
# Example Python code snippet for schema validation using avro and pyarrow
from avro.schema import SchemaFromJSONDict, UnionSchema
from pyarrow import parquet as pq

avro_schema = {
  "name": "data",
  "type": ["null", "int"]
}
parquet_schema = SchemaFromJSONDict({"name": "data", "type": "int"})

# Validation logic would go here to ensure types align properly
```

**Example input / output:**

```

Input Avro schema:

JSON
Copy
{
  "name": "data",
  "type": ["null", "int"]
}

Output Parquet schema (after validation and transformation):

JSON
Copy
{
  "name": "data",
  "type": "int"
}
```

**Data:**

```json
{
  "schema": {
    "fields": [
      {"name": "id", "type": ["null", "long"]},
      {"name": "value", "type": ["null", "float"]}
    ]
  }
}
```

**Flow:**

1. Validated
2. Mismatch Detected
3. Avro Ingest
4. Validate Schemas
5. Transform Data
6. Parquet Output
7. Handle Mismatches

