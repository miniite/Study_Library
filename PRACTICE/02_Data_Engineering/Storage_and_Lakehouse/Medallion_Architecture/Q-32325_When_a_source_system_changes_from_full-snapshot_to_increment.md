---
type: "practice"
difficulty: "Deep"
---

# When a source system changes from full-snapshot to incremental CDC delivery, how do you adapt Bronze ingestion without rewriting all of Silver?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Medallion Layer Quality](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Medallion_Architecture/Medallion_Layer_Quality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the transition from full snapshots to incremental change data capture (CDC) in the context of Medallion Architecture, we need to ensure that our Bronze layer can handle the new data delivery method without significantly impacting the Silver layer. This involves adapting how we ingest and process raw data into a form suitable for analytics while maintaining consistency across layers.

Firstly, let's understand what the question is really asking: We're dealing with an architectural shift in data ingestion from full snapshots to incremental CDC, which requires modifying our Bronze layer’s data processing logic without disrupting the existing Silver layer. The interviewer is probing your ability to manage a seamless transition, considering both technical and operational implications.

#### Triaging the Question
Core Idea

The key idea here is to adapt the Bronze layer's ingestion process to handle incremental changes while ensuring that the downstream Silver layer remains unaffected in terms of data consistency and processing logic.

#### Assumptions
The source system now supports CDC.
The current Bronze layer architecture can be modified incrementally.
The existing Silver layer does not need immediate changes (e.g., it processes already validated and cleaned data).

#### First-Principles Solution
Data Synchronization: Ensure the Bronze layer can recognize new rows being added or updated in a CDC stream without needing to reconstruct all historical records. This typically involves maintaining a transaction log or timestamp that helps identify changes.
Incremental Ingestion Logic: Modify the Bronze ingestion process to only read and process delta data (new/updated rows) from the CDC stream, skipping unchanged full snapshots.
Transformation Consistency: Ensure that any transformations applied in Silver remain consistent even with partial datasets. This might involve maintaining a state of transformation logic or using incremental aggregations.
Detailed Steps

Identify Change Events:

Extract change events (inserts, updates) from the CDC stream.
Store these changes in a temporary staging area within Bronze.

Apply Transformations Incrementally:

Use a batch processing approach to read and apply transformations only to new or updated rows.
Maintain state between batches if necessary to ensure consistency.

Merge Data Efficiently:

Combine the delta data with existing Bronze data using efficient join operations to avoid full scans of large datasets.
Ensure that any full snapshot data from the previous batch is appropriately handled to maintain historical integrity.

#### Closing Summary
and Common Mistakes

In summary, adapting the Bronze layer for incremental CDC involves careful design to handle both new and existing data efficiently. Key challenges include maintaining consistency in transformations and avoiding redundant processing of unchanged records. Common mistakes include not properly handling state across batches or failing to ensure that all changes are accurately reflected without disrupting the Silver layer.

#### Follow-Ups
How do you handle edge cases where the CDC stream is inconsistent?
What measures can be taken to optimize performance during incremental ingestion?
How does this change impact your overall data pipeline architecture?

**Comparison:**

| Key | Bronze Layer Schema |
|-----|---------------------|
| key_column | INT                 |
| field_a      | VARCHAR              |
| field_b      | VARCHAR              |
| Delta Events | CDC Stream Structure |
|--------------|----------------------|
| key_column   | INT                  |
| event_type   | VARCHAR              |
| payload      | JSON                 |

**Code:**

```python
import pandas as pd
from sqlalchemy import create_engine

# Example function for incremental processing in Bronze
def process_incremental_data(cdc_stream, bronze_db_connection):
    # Extract new/updated rows from CDC stream
    changes = cdc_stream.get_new_changes()

    # Load existing data into DataFrame
    existing_data = pd.read_sql("SELECT * FROM bronze_table", bronze_db_connection)

    # Apply transformations only to new/changed records
    transformed_data = apply_transformations(changes, existing_data)

    # Merge transformed data with existing data
    updated_bronze_data = merge_data(existing_data, transformed_data)

    # Store the result back into the Bronze layer
    updated_bronze_data.to_sql('bronze_table', bronze_db_connection, if_exists='replace')

def apply_transformations(changes, existing_data):
    # Example transformation logic (simplified)
    return pd.merge(existing_data, changes, on='key_column', how='outer')

def merge_data(existing_data, transformed_data):
    # Efficiently merge data using SQL JOIN
    with create_engine('postgresql://user:password@localhost/dbname').connect() as conn:
        existing_data.to_sql('temp_table', con=conn, if_exists='replace', index=False)
        query = "MERGE INTO bronze_table AS target USING temp_table AS source ON target.key_column = source.key_column WHEN MATCHED THEN UPDATE SET * WHEN NOT MATCHED THEN INSERT;"
        conn.execute(query)
```

**Example input / output:**

```text
Input:
- CDC Stream: [{key_column: 1, event_type: 'INSERT', payload: {'new_field': 'new_value'}}, {key_column: 2, event_type: 'UPDATE', payload: {'updated_field': 'new_value'}}]
- Bronze Table: [
    {key_column: 1, field_a: 'value_a', field_b: 'value_b'},
    {key_column: 2, field_a: 'old_value', field_b: 'value_b'}
]

Output:
- Updated Bronze Table: [
    {key_column: 1, field_a: 'value_a', new_field: 'new_value', field_b: 'value_b'},
    {key_column: 2, field_a: 'new_value', updated_field: 'new_value', field_b: 'value_b'}
]
```

**Data:**

```json
{
  "cdc_stream": {
    "schema": [
      {"name": "key_column", "type": "INT"},
      {"name": "event_type", "type": "VARCHAR"},
      {"name": "payload", "type": "JSON"}
    ],
    "data": [
      { "key_column": 1, "event_type": "INSERT", "payload": {"new_field": "new_value"} },
      { "key_column": 2, "event_type": "UPDATE", "payload": {"updated_field": "new_value"} }
    ]
  },
  "bronze_table": {
    "schema": [
      {"name": "key_column", "type": "INT"},
      {"name": "field_a", "type": "VARCHAR"},
      {"name": "field_b", "type": "VARCHAR"}
    ],
    "data": [
      { "key_column": 1, "field_a": "value_a", "field_b": "value_b" },
      { "key_column": 2, "field_a": "old_value", "field_b": "value_b" }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Extract New Changes
4. Are Records New/Updated?
5. Merge with Existing Data
6. Skip Record
7. Apply Transformations
8. Store in Bronze Layer

