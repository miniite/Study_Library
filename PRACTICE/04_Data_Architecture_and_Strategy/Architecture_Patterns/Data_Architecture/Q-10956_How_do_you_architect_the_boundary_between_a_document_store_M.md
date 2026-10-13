---
type: "practice"
difficulty: "Deep"
---

# How do you architect the boundary between a document store (MongoDB/DynamoDB) as system of record and a columnar analytical lake, given schema flexibility on the operational side? Where does schema-on-read at the lake become a liability when the document shape drifts silently across releases?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Warehouse Vendor Exit Strategy](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Layered_Pipelines_Bronze_Silver_Gold/Warehouse_Vendor_Exit_Strategy.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to architect a data pipeline that ensures seamless integration between a highly flexible document store like MongoDB or DynamoDB (acting as the system of record) and a columnar analytical lake, such as Apache Parquet or ORC files stored in HDFS. The key challenge is maintaining schema consistency across both systems while minimizing drift issues during application releases.

Firstly, let's break down the question: We are dealing with two different data models—document-oriented and columnar. Document stores like MongoDB/DynamoDB support complex nested structures and dynamic schemas, making them ideal for operational workloads where data flexibility is crucial. On the other hand, analytical lakes are optimized for read-heavy operations on large datasets, often using schema-on-read to accommodate diverse data types without upfront constraints.

To address the drift issue, we need a robust approach that ensures changes in the document store's schema do not propagate silently into the analytics pipeline, leading to errors or unexpected behavior. This involves designing a system of record with versioned schemas and carefully managed schema transformations between systems.

Schema Versioning: Implement schema versioning in your document store. Each release should include a clear definition of the expected data structure, which can be stored alongside application code or version-controlled within Git. This ensures that both developers and the data pipeline are aware of what changes are being made.

Incremental Data Pipelines: Use incremental extraction, transformation, and loading (ETL) processes to transfer only relevant changes from the document store to the analytical lake. Techniques like CDC (Change Data Capture) can help in identifying updates efficiently.

Schema Validation at Ingestion: Implement schema validation during data ingestion into the analytical lake. Tools like Apache Beam or Spark can be used to enforce constraints based on the current schema version, ensuring that any incompatible data is rejected before processing.

Metadata Management: Maintain metadata about schema changes in a separate service or database. This helps track schema evolution and enables automated reconciliation between different systems. For example, you could use Airbyte for ETL jobs with built-in validation logic.

Testing and CI/CD Integration: Integrate schema checks into your continuous integration (CI) pipelines to automatically validate data transformations during development cycles. This ensures that potential issues are caught early in the release cycle.

Drift Detection and Retrospective Fixes: Implement mechanisms for detecting schema drift either through manual audits or automated monitoring tools like Databend or DataHub, which can alert stakeholders when unexpected changes occur. These alerts should trigger retrospective fixes to ensure data integrity.

The overall architecture would look something like this:

Document Store (MongoDB/DynamoDB): Stores raw, flexible operational data.
ETL Pipeline: Transforms and loads selected data into the analytical lake based on schema validation rules.
Analytical Lake (Parquet/ORC in HDFS): Stores transformed and validated data for querying and analysis.
Monitoring and Validation Layer: Monitors schema changes and ensures consistency across systems.

By following these steps, we can maintain a balance between operational flexibility and analytical accuracy, reducing the risk of silent schema drift causing issues downstream. This approach not only preserves data integrity but also enhances the overall efficiency of the data architecture.

**Comparison:**

```
Schema Version	Fields	Constraints
v1.0	id, first_name	minLength: 3
v2.0	customer_id, product_id	min: 1, required
```

**Code:**

```python
# Example ETL Pipeline using Apache Beam for incremental extraction and validation

def process_records(records):
    # Apply schema validation logic here
    if validate_record(record, current_schema_version):
        return record  # Valid record to be processed further
    else:
        raise ValueError("Record does not conform to the expected schema")

# Pseudo-code for CDC-based ETL job
with beam.Pipeline(options=options) as p:
    records = (
            p
            | 'Read From MongoDB' >> ReadFromMongoDB()
            | 'Validate Records' >> beam.Map(process_records)
            | 'Write To Lake' >> WriteToParquetLake(current_schema_version)
    )
```

**Example input / output:**

```text
Input (MongoDB Document):
{
  "_id": ObjectId("605f4a7b98d9e1bda2a2c0ff"),
  "customer_id": "C001",
  "first_name": "John",
  "last_name": "Doe"
}

Validation Output:
- Valid document (no errors)

Input (MongoDB Document with missing field):
{
  "_id": ObjectId("605f4a7b98d9e1bda2a2c0ff"),
  "customer_id": "C001",
  "last_name": "Doe"
}

Validation Output:
- Invalid document: Missing required field 'first_name'
```

**Data:**

```json
{
  "current_schema_version": "v2.3",
  "schema_definitions": [
    {
      "name": "customer_profile",
      "version": "v1.0",
      "fields": [
        { "name": "id", "type": "string" },
        { "name": "first_name", "type": "string" }
      ]
    },
    {
      "name": "order_details",
      "version": "v2.0",
      "fields": [
        { "name": "id", "type": "int64" },
        { "name": "customer_id", "type": "string" },
        { "name": "product_id", "type": "string" }
      ]
    }
  ],
  "validation_rules": {
    "rules": [
      {"field": "id", "constraints": [{"required": true}]},
      {"field": "first_name", "constraints": [{"minLength": 3}]}
    ]
  }
}
```

**Flow:**

1. Document Store(MongoDB/DynamoDB)
2. ETL Pipeline
3. Analytical Lake (Parquet/ORC)
4. Metadata Management &Validation Service
5. Monitoring and Alerting

