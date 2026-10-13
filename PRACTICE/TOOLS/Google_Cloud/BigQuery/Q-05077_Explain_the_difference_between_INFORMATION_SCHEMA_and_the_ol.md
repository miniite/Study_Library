---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between INFORMATION_SCHEMA and the older __TABLES__ and Data Catalog for metadata, and how you would build a lineage/discovery layer.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ BigQuery Loading and Table Design](<../../../../TOOLS/Google_Cloud/BigQuery/BigQuery_Loading_and_Table_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain the differences between INFORMATION_SCHEMA, the older __TABLES__ view, and BigQuery's Data Catalog in terms of metadata management, we need to first understand their roles and limitations. The core concept is that while INFORMATION_SCHEMA provides a simple way to access schema information, it has several limitations compared to the more advanced capabilities offered by the Data Catalog. Additionally, lineage/discovery layers are crucial for tracking data flow and ensuring data quality.

#### Triage

The question probes your understanding of BigQuery's metadata tools and how they can be leveraged to build robust lineage and discovery systems. You need to explain the key differences between INFORMATION_SCHEMA and the older __TABLES__, discuss the limitations of these views, and then describe how you would use the Data Catalog for building a lineage/discovery layer.

Core Concept Recap
INFORMATION_SCHEMA: A built-in view that provides schema information.
TABLES: An older view that was available before INFORMATION_SCHEMA became standard.
Data Catalog: A more advanced metadata management system provided by BigQuery.

#### First-Principles Solution
INFORMATION_SCHEMA

INFORMATION_SCHEMA is a built-in view in SQL databases, including BigQuery. It provides information about the schema of tables and columns within your database. This includes details like table names, column names, data types, etc. Here’s an example query:

SQL
SELECT TABLE_NAME, COLUMN_NAME, DATA_TYPE
FROM `project_id.dataset.INFORMATION_SCHEMA.COLUMNS`

Limitations:

Limited Metadata: It only provides basic schema information.
Static Information: It does not capture dynamic changes in the data or schema over time.
TABLES

__TABLES__ is an older view that was used before INFORMATION_SCHEMA became standard. It served a similar purpose but had limitations:

Deprecated: Now considered deprecated and less reliable.
Limited Functionality: Similar to INFORMATION_SCHEMA, it only provides static schema information.
Data Catalog

BigQuery's Data Catalog is a more advanced metadata management tool that offers several advantages over INFORMATION_SCHEMA and __TABLES__:

Dynamic Metadata: It captures real-time changes in data and schema.
Enhanced Discoverability: Facilitates easy discovery of datasets, tables, and columns.
Lineage Tracking: Provides lineage information to track how data is used throughout the organization.
Building a Lineage/Discovery Layer

To build a lineage/discovery layer using BigQuery's Data Catalog:

Register Metadata: Register your dataset and table metadata in the Data Catalog.
Capture Changes: Use Data Catalog’s change tracking features to capture schema changes dynamically.
Lineage Tracing: Trace data lineage by following how data moves through various transformations and storage layers.
Steps to Implement

Dataset Registration:

Register datasets and tables in the Data Catalog using bq metadata command or API calls.
PYTHON
bq --location=US metadata register dataset_id project_id

Schema Changes:

Monitor schema changes using BigQuery’s event triggers and update Data Catalog accordingly.
SQL
CREATE OR REPLACE SCHEMA table_name.column_name AS new_data_type;

Lineage Tracing:

Integrate with other tools like Apache Beam or Spark for data processing to log lineage information.
Use BigQuery’s INFORMATION_SCHEMA.JOBS view to trace job history and lineage.

#### Example
Workflow
PYTHON
import google.cloud.bigquery as bq

# Register dataset in Data Catalog
client = bq.Client()
dataset_ref = client.dataset('my_dataset')
client.create_dataset(dataset_ref)

# Monitor schema changes
def on_schema_change(event):
    if isinstance(event, bq.SchemaChangedEvent):
        table_ref = event.table_reference
        client.update_table(table_ref, ["schema"])

event_subscriber = client.event_subscriber
event_subscriber.subscribe(on_schema_change)

#### Closing Summary
and Common Mistakes

In summary, while INFORMATION_SCHEMA and __TABLES__ provide basic schema information, they lack the dynamic capabilities and advanced features offered by BigQuery's Data Catalog. For building a robust lineage/discovery layer, leveraging Data Catalog is essential as it supports real-time metadata management and comprehensive lineage tracking.

#### Common Mistakes

Over-relying on static INFORMATION_SCHEMA or deprecated __TABLES__.
Failing to integrate dynamic monitoring of schema changes.
Not utilizing BigQuery’s event triggers for automated lineage tracing.

**Code:**

```python
import google.cloud.bigquery as bq

client = bq.Client()
dataset_ref = client.dataset('my_dataset')
client.create_dataset(dataset_ref)

def on_schema_change(event):
    if isinstance(event, bq.SchemaChangedEvent):
        table_ref = event.table_reference
        client.update_table(table_ref, ["schema"])

event_subscriber = client.event_subscriber
event_subscriber.subscribe(on_schema_change)
```

