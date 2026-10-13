---
type: "practice"
difficulty: "Expert"
---

# How would you architect metadata capture so it survives an engine migration — say Redshift to Snowflake to Databricks over several years — without re-implementing lineage extraction each time?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Catalog Consistency Model](<../../../../05_Data_Governance_and_Security/Governance/Catalog_and_Metadata/Catalog_Consistency_Model.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that metadata capture and lineage information survive a multi-year engine migration from Redshift, Snowflake, to Databricks without needing to re-implement lineage extraction each time, we must adopt a robust approach that includes centralized storage, version-controlled schemas, and a comprehensive data catalog. This solution ensures that lineage and metadata are captured once and remain accessible throughout the migration process.

The key idea is to design an architecture that decouples metadata capture from specific data processing engines. By doing so, you can maintain a consistent metadata model across different environments while ensuring its persistence during migrations.

#### Triaging the Question

This question probes your ability to handle complex system transformations and understand how to maintain consistency in metadata management over time. It also requires you to think about long-term data governance strategies that ensure lineage is captured and preserved, regardless of the underlying technology stack changes. Key assumptions include:

The engines will undergo multiple versions or types (e.g., Redshift to Snowflake to Databricks).
Lineage information must remain accurate and accessible throughout these migrations.
There are no specific constraints on storage capacity or performance.

#### First-Principles Solution

To architect a metadata capture system that survives engine migrations, follow these steps:

Centralized Metadata Repository:

Design a centralized metadata repository to store lineage data, schema definitions, and other metadata in a consistent format.
Use a scalable storage solution like AWS S3, Google Cloud Storage, or Azure Blob Storage.

Version-Controlled Schema Definitions:

Maintain version-controlled schema definitions for each engine type (e.g., Redshift, Snowflake, Databricks).
Use tools like Git to manage these schemas and ensure changes are tracked and reversible.

Metadata Extraction Layer:

Implement a metadata extraction layer that captures lineage data directly from the engines.
Ensure this layer can adapt to different query languages and data models supported by each engine.

Persistent Storage of Metadata:

Store the extracted metadata in the centralized repository, ensuring it is versioned and easily accessible.
Use JSON or a structured format that supports lineage information such as JSON-LD or custom metadata structures.

Data Catalog Integration:

Integrate the centralized metadata into a data catalog (e.g., Looker, Superset) to provide an interface for querying and managing lineage.
Ensure the data catalog is updated with new schema definitions and lineage data as engines are migrated.

Automation and Orchestration:

Automate the process of extracting metadata from each engine during migrations.
Use orchestration tools like Apache Airflow or AWS Step Functions to manage the workflow and ensure it runs smoothly.

#### Closing Summary

By implementing a centralized, version-controlled metadata repository and ensuring consistent extraction and storage practices, you can maintain lineage information across different engines without needing to re-implement extraction logic. This approach ensures that your data governance strategies remain robust and effective even as technology stacks evolve.

#### Common Mistakes/Follow-Ups

Failing to consider the long-term maintenance of metadata repositories.
Not accounting for versioning in schema definitions, leading to inconsistencies during migrations.
Overlooking automation tools that can streamline the migration process.

**Comparison:**

| Engine Type | Schema Definitions | Metadata Storage |
|-------------|--------------------|-----------------|
| Redshift    | JSON               | S3              |
| Snowflake   | JSON               | S3              |
| Databricks  | JSON               | S3              |

**Code:**

```python
# Example Python code snippet to illustrate metadata extraction and storage

import json
from datetime import datetime

def extract_metadata(engine_type, query_result):
    # Extract relevant lineage data from query result based on engine type
    if engine_type == 'Redshift':
        schema = query_result['schema']
        table_name = query_result['table']
        columns = [col['name'] for col in query_result['columns']]
    elif engine_type == 'Snowflake':
        # Extract metadata for Snowflake
        pass
    elif engine_type == 'Databricks':
        # Extract metadata for Databricks
        pass

    lineage_info = {
        "timestamp": datetime.now().isoformat(),
        "schema": schema,
        "table_name": table_name,
        "columns": columns
    }

    return lineage_info

def store_metadata(metadata, repository):
    # Store the extracted metadata in the centralized repository
    with open(repository, 'a') as file:
        json.dump(metadata, file)
        file.write('\n')
```

**Example input / output:**

```text
Input:
- Redshift query result with schema, table name, and columns

Output:
{
  "timestamp": "2023-10-05T14:48:00",
  "schema": "public",
  "table_name": "sales_data",
  "columns": ["id", "product_id", "customer_id", "quantity_sold", "date_of_sale"]
}
```

**Data:**

```json
{
  "metadata": {
    "timestamp": "2023-10-05T14:48:00",
    "schema": "public",
    "table_name": "sales_data",
    "columns": ["id", "product_id", "customer_id", "quantity_sold", "date_of_sale"]
  }
}
```

**Flow:**

1. No
2. Yes
3. Engine Query
4. Is Engine Changing?
5. Extract Metadata
6. Store in Repository
7. Migrate Metadata to NewEngine
8. Extract and Store MetadataContinuously

