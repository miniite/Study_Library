---
type: "practice"
difficulty: "Deep"
---

# How do you keep the catalog and the actual data estate from drifting — detecting orphaned entries, tables that exist but were never ingested, and ghost lineage to deleted assets — as a continuous reconciliation process?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Catalog Consistency Model](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Catalog_Consistency_Model.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a data catalog remains in sync with the actual data estate, we need to implement a robust, continuous reconciliation process. This involves detecting orphaned entries, tables that exist but were never ingested, and ghost lineage to deleted assets. The core idea is to maintain integrity by regularly comparing metadata from the catalog against the actual state of the data stores.

First, let's triage the question. The interviewer is probing how you manage metadata consistency in a complex data ecosystem. We need to consider assumptions such as the presence of multiple data sources and a centralized or distributed data storage architecture. The solution will involve technical details like schema comparison, lineage tracking, and automated reconciliation mechanisms.

Starting from first principles:

Metadata Collection: Gather comprehensive metadata about all assets (tables, columns, schemas) in both the catalog and the actual data stores.
Schema Comparison: Compare the schema definitions between the catalog and the actual data stores to detect discrepancies.
Lineage Tracking: Ensure that lineage information is maintained and can be traced back from end-to-end.
Reconciliation Mechanisms: Implement real-time or periodic checks to reconcile any differences found during schema comparison.

To implement this, we will:

Use a data catalog system like Apache Atlas, AWS Glue, or custom-built solutions.
Employ a lineage management tool that tracks dependencies and updates automatically.
Set up automated scripts or CI/CD pipelines for continuous monitoring.
Integrate with version control systems to manage schema changes.

Let's walk through each step in more detail:

Metadata Collection:

Collect metadata from all data stores using APIs, ETL processes, or direct database introspection.
Store the collected metadata in a centralized catalog system for easy querying and comparison.

Schema Comparison:

Define a schema model that captures structure (tables, columns), relationships, and business rules.
Write a script or use a tool like DBT (Data Build Tool) to compare the models from both the catalog and the data stores.
Highlight differences such as missing tables, columns with different types, or discrepancies in constraints.

Lineage Tracking:

Implement a system that tracks how data moves through the pipeline, including transformations, joins, and aggregations.
Ensure lineage is captured at both a high level (business processes) and a granular level (specific code segments).
Use tools like Apache Airflow to manage workflows and automatically update lineage metadata.

Reconciliation Mechanisms:

Schedule periodic checks using tools like cron jobs or managed services.
Trigger alerts for any detected discrepancies, providing context about the affected assets.
Automate remediation steps if feasible (e.g., delete ghost entries, correct schema mismatches).

In summary, keeping a data catalog in sync with the actual data estate requires a multifaceted approach that involves metadata collection, schema comparison, lineage tracking, and automated reconciliation. Regular monitoring and proactive management are key to maintaining consistency.

#### Common mistakes
include:

Failing to implement continuous monitoring leading to delayed detection of issues.
Overlooking lineage tracking which can result in incorrect or incomplete understanding of data transformations.
Not automating remediation steps, leaving manual intervention as the only option.

**Comparison:**

| Column | Type    |
|--------|---------|
| Table  | String  |
| Column | String  |
| CurrentType | String  |
| ActualType | String  |
| Table      | Column       | CurrentType | ActualType   |
|------------|--------------|-------------|--------------|
| table2     | address      | VARCHAR(50) | VARCHAR(100) |
|            | city         | VARCHAR(30) | VARCHAR(40)  |

**Code:**

```python
# Example script for schema comparison using DBT
from dbt.utils import parse_file

def compare_schemas(catalog_model, database_model):
    catalog_columns = parse_file(catalog_model)
    dbt_columns = parse_file(database_model)

    missing_tables = [table for table in catalog_columns if table not in dbt_columns]
    extra_tables = [table for table in dbt_columns if table not in catalog_columns]

    mismatches = []
    for table in set(catalog_columns.keys()).intersection(dbt_columns.keys()):
        catalog_cols = catalog_columns[table]['columns']
        dbt_cols = dbt_columns[table]['columns']

        for col in catalog_cols:
            if col not in dbt_cols or dbt_cols[col] != catalog_cols[col]:
                mismatches.append((table, col, catalog_cols[col], dbt_cols[col]))

    return missing_tables, extra_tables, mismatches
```

**Example input / output:**

```text
# Sample input and output for the schema comparison script
catalog_model = parse_file("path/to/catalog/schema.json")
database_model = parse_file("path/to/database/schema.json")

missing_tables, extra_tables, mismatches = compare_schemas(catalog_model, database_model)
print(f"Missing Tables: {missing_tables}")
print(f"Extra Tables: {extra_tables}")
print(f"Mismatches: {mismatches}")
```

**Data:**

```json
{
  "catalog_schema": {
    "table1": {
      "columns": ["id", "name", "email"]
    },
    "table2": {
      "columns": ["id", "address", "city"]
    }
  },
  "database_schema": {
    "table1": {
      "columns": ["id", "name", "email"]
    },
    "table3": {
      "columns": ["id", "zip_code", "state"]
    }
  }
}
```

**Flow:**

1. Metadata Collection
2. Schema Comparison
3. Lineage Tracking
4. Reconciliation Mechanisms
5. Periodic Checks
6. Remediation Alerts
7. Automated Scripts

