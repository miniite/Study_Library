---
type: "practice"
difficulty: "Expert"
---

# How do you keep a schema registry and a downstream Parquet warehouse schema in sync when the two evolve on different cadences?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Avro Schema Evolution](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Avro_Schema_Evolution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a schema registry and a downstream Parquet warehouse remain in sync, especially when they evolve at different rates, we need to establish a robust process that allows for seamless integration and conflict resolution. This involves understanding how schemas are registered and changed, defining an automated or semi-automated update strategy, and handling schema migrations gracefully.

The question is probing your ability to manage data schema evolution in a scalable and reliable manner. Key assumptions include:

The schema registry manages versioned schemas.
Parquet files store the current version of the schema according to the metadata embedded within them.
You have control over the ingestion pipeline that writes to the Parquet warehouse.

We will reason step-by-step from first principles, starting with a foundational understanding and progressing to a full solution:

Understanding Schema Evolution: In a schema registry like Apache Avro or an external service such as Confluent Schema Registry, schemas are versioned and tracked over time. Each update to the schema increments a version number.

Schema Versioning in Parquet: Parquet files store their schema metadata directly within the file structure. When writing data to Parquet, the current schema is embedded, allowing for future readers to understand how to interpret the data.

Conflict Resolution Strategies:

Incremental Updates: Continuously monitor the schema registry and push updates to the Parquet warehouse.
Schema Compatibility: Ensure that new versions of schemas are backward compatible or handle breaking changes through transformations (e.g., renaming fields, adding optional columns).

Automated vs. Manual Updates:

Automated: Use a script or tool to periodically fetch the latest schema from the registry and update the Parquet warehouse.
Manual: Introduce a manual review process before applying changes, which is less scalable but more controlled.

Handling Inconsistent Data:

Backfilling: For data that doesn’t align with new schemas, implement backfill logic to convert older formats into the latest schema.
Error Handling: Design error handling mechanisms to manage scenarios where the Parquet warehouse and registry are out of sync.

Monitoring and Validation:

Regularly check for discrepancies between the registry and warehouse using tools or scripts.
Implement health checks and alerts when schema mismatches are detected.

Best Practices:

Use version control to manage changes in both the registry and warehouse.
Document all schema updates, including reasons for each change.
Perform thorough testing before applying any schema changes.

By following these steps, we can ensure that the schema registry and Parquet warehouse remain synchronized even when they evolve at different rates.

**Comparison:**

| Schema Version | Fields         |
|----------------|----------------|
| 1              | id (int)       |
| 2              | name (string)  |
| 3              | age (int)      |
| 4              | email (string) |

**Code:**

```python
# Example Python script to fetch latest schema from registry and update Parquet warehouse
import requests

def get_latest_schema(registry_url):
    response = requests.get(f"{registry_url}/latest")
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception("Failed to fetch the latest schema")

def update_parquet_warehouse(schema, parquet_path):
    # Code to write schema metadata into Parquet file or create new versioned files
    pass

latest_schema = get_latest_schema("http://schema-registry.example.com")
update_parquet_warehouse(latest_schema, "/path/to/parquet/warehouse")
```

**Example input / output:**

```text
Input:
- Latest schema from registry

Output:
- Updated Parquet files with the latest schema metadata

Dry-run:
1. Fetch latest schema version 4.
2. Apply changes to existing Parquet warehouse or backfill data as needed.

Request:
- Ensure that the parquet_warehouse/000001.parquet contains the correct schema version and metadata.

Response:
- The updated file includes the new schema fields and compatibility handling if necessary.
```

**Data:**

```json
{
  "schema_id": 123,
  "version": 4,
  "fields": [
    {"name": "id", "type": "int"},
    {"name": "name", "type": "string"}
  ]
}
```

**Flow:**

1. Schema Registry
2. Fetched Latest Schema
3. Update Parquet Warehouse
4. Backfill Data if Necessary
5. Validate Schema Sync

