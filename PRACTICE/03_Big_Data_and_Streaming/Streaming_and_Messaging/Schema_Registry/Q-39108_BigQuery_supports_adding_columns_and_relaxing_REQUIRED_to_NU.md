---
type: "practice"
difficulty: "Deep"
---

# BigQuery supports adding columns and relaxing REQUIRED to NULLABLE but not much else. How do you design ingestion so schema drift doesn't force full reloads?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Schema IDs and Compaction](<../../../../03_Big_Data_and_Streaming/Streaming_and_Messaging/Schema_Registry/Schema_IDs_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To handle schema evolution in BigQuery without forcing full data reloads, we need a robust approach that leverages schema registry and versioning techniques. This ensures that our ingestion pipeline can adapt to changes in the schema while minimizing downtime and storage overhead.

#### Triage
Understand the Question: The core of this question is about managing schema drift in BigQuery without triggering full data reloads, which are expensive in terms of both time and resources.
Identify Key Assumptions:
We need to support adding new columns, relaxing REQUIRED fields to NULLABLE, and possibly removing columns.
The ingestion process should be flexible enough to handle schema changes without reprocessing the entire dataset.
Key Ideas: Use a schema registry and versioning strategy to track and manage schema changes.

#### First-Principles Solution
Schema Registry: Implement a central repository where all schema versions are stored. This allows us to maintain a history of how the schema has evolved over time.
Versioned Ingestion Pipeline:
The pipeline should be designed to handle different schema versions based on the incoming data's metadata (e.g., timestamp or version number).
For new columns, we can append them in subsequent batches or update the schema incrementally.
Handling Schema Migrations: Develop a script or tool that can migrate existing data to the latest schema if necessary. This might involve filling default values for newly added NULLABLE fields.
Detailed Solution:

Schema Versioning:

Store each version of your schema in a dedicated table or directory within BigQuery.
Use a version number (e.g., v1, v2) to identify the current and historical schemas.

Metadata Handling:

Include metadata with each record that indicates which schema version it adheres to.
This could be done by adding a schema_version column in the source data or tagging records appropriately.

Ingestion Process:

When ingesting new data, the pipeline should first validate the incoming data against the current schema version.
For changes like adding columns, the pipeline can append these to existing tables without issues.
For relaxing REQUIRED fields to NULLABLE, ensure that old data is updated appropriately before it’s stored in BigQuery.

Schema Migration Tool:

Develop a script or tool that can automatically migrate older data to match new schema versions if required.
This tool should be able to handle different types of schema changes (adding columns, relaxing REQUIRED fields, etc.).

Testing and Validation:

Regularly validate the schema against historical data to ensure there are no inconsistencies.
Use a staging environment for applying schema migrations before moving changes to production.

#### Closing Summary
+ Common Mistakes:
Common Mistake: Ignoring schema evolution can lead to significant downtime when full reloads become necessary. Proper versioning and handling of metadata can prevent this.
Best Practice: Always have a plan in place for schema migration, including both incremental updates and full migrations as needed.

#### Example Scenario

#### Cons
ider a scenario where we add a new column new_field to our schema. The ingestion pipeline would first validate the incoming data against the old schema. If all records are valid, it can proceed without issues. However, if there are missing fields, the tool will populate default values or handle them appropriately.

Common Follow-Ups:
How do you manage conflicts between different schema versions?
What measures do you take to ensure data consistency during migrations?

**Comparison:**

```
Schema Version	Columns
v1	col1, col2
v2	col1, col2, new_field
```

**Code:**

```python
# Example Python script for migrating data
def migrate_data(old_schema, new_schema):
    # Logic to handle adding or updating columns based on the difference in schemas
    pass

def validate_data(record, schema_version):
    # Validate record against the specified schema version
    pass
```

**Example input / output:**

```
Input:
JSON
Copy
[
  {"col1": "value1", "col2": "value2"},
  {"col1": "value3"}
]
Output:
For v1 schema, records are ingested as-is.
For v2 schema, a new field new_field is added with default values where necessary.
```

**Data:**

```json
{
  "schema_versions": [
    {"version": "v1", "columns": ["col1", "col2"]},
    {"version": "v2", "columns": ["col1", "col2", "new_field"]}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Ingest Data
4. Validate Metadata
5. Update Schema Version
6. Store Data
7. Migrate Data to New Schema

