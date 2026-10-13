---
type: "practice"
difficulty: "Intermediate"
---

# What happens to your pipeline when a new column is added to the source table?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Managed Ingestion and Re-syncs](<../../../../02_Data_Engineering/Pipelines_and_Integration/Data_Ingestion/Managed_Ingestion_and_Re-syncs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a new column is added to the source table, the data ingestion process can be disrupted or altered in various ways depending on how it is handled by tools like Airbyte and Fivetran. In this response, I'll explain the key steps involved, starting from the basics of what happens under the hood.

First, let's clarify that both Airbyte and Fivetran are widely used data integration tools. They operate by establishing connections to source databases (like MySQL or PostgreSQL) and replicating their contents into target systems like a data warehouse (e.g., BigQuery). The core concept here is that these tools need to be aware of changes in the schema of the source tables, including new columns.

#### Triage
Airbyte uses a delta sync approach by default for incremental updates, while Fivetran generally relies on full refreshes or change capture. We'll consider both approaches and their implications when a new column is introduced.

Step-by-step solution:

Schema Detection: The first step involves detecting changes in the schema of the source table. Airbyte and Fivetran must identify that a new column has been added to the table. This is typically done by periodically querying metadata APIs or running introspection queries on the database.

Configuration Update:

In Airbyte, this would require updating the catalog.json configuration file to include the new column.
For Fivetran, the schema needs to be manually updated in the service settings if it's a one-time setup, or automatically via their change detection mechanisms.

Data Replication:

If Airbyte detects that a new column is present and not part of its existing schema, it will update its internal state to include this new field.
Fivetran might handle the schema changes more gracefully if they support automated updates or if you manually trigger an update.

Data Ingestion:

When Airbyte processes a batch of data from the source table, it now includes values for the newly added column in each record.
For Fivetran, depending on whether it supports dynamic schema changes, this might require a full refresh or partial sync to include the new column.

Data Validation:

Ensure that the new column is properly mapped and validated against the target schema. This can involve checks to ensure data types match and nullability constraints are respected.

Error Handling:

If there are issues with the new column (e.g., missing values or invalid formats), Airbyte/Fivetran should handle these gracefully by logging errors, retrying operations, or providing alerts.

#### Closing summary
When a new column is added to the source table, data ingestion pipelines like those managed by Airbyte and Fivetran need to adapt by updating their configurations and ensuring that the data from the new column is processed correctly. This process involves schema detection, configuration updates, data replication, validation, and error handling.

#### Common mistakes/follow-ups

Overlooking the need for manual intervention in Fivetran if not set up with automated schema changes.
Not considering edge cases like invalid values in the new column that could cause ingestion failures.
Ignoring performance impacts of full refreshes versus incremental updates when dealing with large datasets.

**Comparison:**

    | Column Name | Data Type     |
    |-------------|--------------|
    | id          | integer      |
    | name        | string       |
    | new_column_name | string      |

**Code:**

```python
# Example Airbyte configuration update for a new column
import json

catalog = {
    "streams": [
        {
            "tap_stream_id": "example_table",
            "schema": {
                "properties": {
                    # existing columns here ...
                    "new_column_name": {"type": ["null", "string"]}
                }
            },
            # other stream details...
        }
    ]
}

with open('catalog.json', 'w') as f:
    json.dump(catalog, f)
```

**Example input / output:**

```text
Input:
- Source Table before: id, name
- Source Table after adding new column: id, name, new_column_name

Output:
- New records will include 'null' for the `new_column_name` field if it's not populated.
- Existing records will have null values in `new_column_name` unless backfilled or updated manually.

Dry Run:
| id | name  | new_column_name |
|----|-------|----------------|
| 1  | Alice | None           |
| 2  | Bob   | Null           |
```

**Data:**

```json
{
  "table_name": "example_table",
  "columns": [
    {"name": "id", "type": "integer"},
    {"name": "name", "type": "string"},
    {"name": "new_column_name", "type": "string"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. Yes
5. No
6. Source Schema ChangeDetected
7. Is Airbyte?
8. Update Catalog.json
9. Replicate Data with NewColumn
10. Is Fivetran?
11. Check for Automated Updates
12. Automatically Update Schema& Sync
13. Manually Trigger FullRefresh/Incremental Update
14. Validate Data Ingestion
15. Handle Errors (Logging,Retries, Alerts)

