---
type: "practice"
difficulty: "Deep"
---

# How do you approach loading semi-structured API responses whose schema evolves weekly, keeping historical queries stable?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Handling Source Changes and Spikes](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Handling_Source_Changes_and_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with semi-structured API responses that evolve weekly, a critical challenge is maintaining historical data stability while adapting to new schema changes. The key approach involves a combination of schema evolution strategies, data validation, and incremental loading techniques. Let's break down the process from first principles.

#### Triaging the Question

The core of this question revolves around handling semi-structured data (often JSON or XML) with an evolving schema in an ETL/ELT pipeline. The interviewer is probing your understanding of schema evolution strategies, how to handle incremental updates without disrupting historical data, and ensuring that existing queries continue to produce consistent results despite schema changes.

#### Assumptions
Historical Data: You need to maintain the integrity of existing data, i.e., historical records should not be altered.
Schema Evolution: The API response schema changes weekly, but the core data fields and their relationships remain relatively stable over time.

#### Performance Considerations
Performance impact due to handling evolving schemas needs to be minimized.
Step-by-Step Solution

Data Ingestion and Parsing:

Use tools like Apache NiFi or Airbyte for real-time or batch ingestion of API responses.
Parse semi-structured data into a structured format (e.g., JSON, Avro) using libraries such as Jackson or schematizer.

Schema Versioning:

Implement schema versioning to track historical versions of the schema. This can be done by storing schema metadata in a database table or using an external service like AWS Schema Registry.
For each new weekly update, create a new schema version and map it to existing data where possible.

Incremental Loading:

Use incremental loading techniques such as upserts (update-if-exists) to handle partial updates without rewriting historical records.
Leverage database features like CDC (Change Data Capture) or watermarking to track changes in real-time and apply them incrementally.

Validation and Transformation:

Validate incoming data against the current schema version using tools like Apache Avro or JSON Schema.
Perform necessary transformations to align new data with existing data models, ensuring consistency across versions.

Historical Data Integrity:

Ensure that historical queries continue to produce consistent results by using materialized views or snapshots of the data at different points in time.
For complex queries, use logical databases or temporary tables to maintain stability during schema changes.

Monitoring and Alerting:

Set up monitoring for data quality and performance metrics.
Configure alerting to notify stakeholders about potential issues during schema migrations.

#### Closing Summary

Maintaining historical data integrity while handling evolving schemas in a semi-structured API scenario requires a multi-faceted approach involving versioned schemas, incremental loading techniques, data validation, and consistent query results. By implementing these strategies, you can ensure that your ETL/ELT pipeline remains robust and scalable.

#### Common Mistakes/Follow-Ups
Overwriting Historical Data: Always use upserts or merge statements to avoid accidentally overwriting historical records.
Inconsistent Queries: Ensure that queries are tested against different schema versions to catch any discrepancies early.
Performance Bottlenecks: Monitor performance closely, especially during large schema changes, and optimize as necessary.

**Comparison:**

| Schema Version | Fields Added | Fields Removed | Changes Mapped |
|----------------|--------------|----------------|----------------|
| V1             | None         | None           | None           |
| V2             | order_date   | amount (renamed) | yes            |

**Code:**

```python
# Example of using Apache Avro for parsing semi-structured data
from avro.datafile import DataFileReader
from avro.io import DatumReader

with open('semi_structured_data.avro', 'rb') as file:
    reader = DataFileReader(file, DatumReader())
    for record in reader:
        print(record)
```

**Example input / output:**

```text
Input JSON:
{
  "customer": {
    "id": "12345",
    "name": "John Doe",
    "address": {
      "street": "123 Main St",
      "city": "Anytown"
    },
    "orders": [
      {"date": "2023-01-01", "amount": 100},
      {"date": "2023-02-01", "amount": 200}
    ]
  }
}

Output JSON (transformed and validated):
{
  "customer_id": "12345",
  "name": "John Doe",
  "street": "123 Main St",
  "city": "Anytown",
  "orders": [
    {"order_date": "2023-01-01", "amount": 100},
    {"order_date": "2023-02-01", "amount": 200}
  ]
}
```

**Data:**

```json
{
  "customer": {
    "id": "12345",
    "name": "John Doe",
    "address": {
      "street": "123 Main St",
      "city": "Anytown"
    },
    "orders": [
      {"date": "2023-01-01", "amount": 100},
      {"date": "2023-02-01", "amount": 200}
    ]
  }
}
```

**Flow:**

1. Ingest API Response
2. Parse Semi-Structured Data
3. Validate Against CurrentSchema
4. Mapped SchemaTransformation
5. Incremental Load intoHistorical Tables
6. Materialized Views forConsistent Queries
7. Monitoring and Alerting

