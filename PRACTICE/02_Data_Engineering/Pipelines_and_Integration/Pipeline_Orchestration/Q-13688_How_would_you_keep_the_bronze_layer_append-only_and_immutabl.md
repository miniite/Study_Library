---
type: "practice"
difficulty: "Expert"
---

# How would you keep the bronze layer append-only and immutable while still supporting GDPR deletions that must reach the gold aggregates?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ GDPR Deletion in Feature Pipelines](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/GDPR_Deletion_in_Feature_Pipelines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the requirement of keeping the bronze layer in an append-only, immutable state while supporting GDPR deletions that propagate to the gold aggregates, we need a robust design that ensures data integrity and compliance with privacy regulations. This solution involves creating a two-tier system where the bronze layer is strictly append-only, and changes are managed through a delta-layer that allows for updates and deletions, which then get applied to the gold aggregate layers.

**Comparison:**

```
Layer	Schema
Bronze	id, data_column, timestamp
Delta	id, action (INSERT, UPDATE, DELETE)
Gold	id, aggregated_data_column
```

**Code:**

```python
# Define the Airflow DAG structure
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'data_engineer',
    'start_date': datetime(2023, 1, 1),
}

dag = DAG(
    'bronze_to_gold_with_gdpr',
    default_args=default_args,
    schedule_interval='@daily',
)

def bronze_to_delta(**kwargs):
    # Process data and update the delta layer
    pass

def delta_to_gold(**kwargs):
    # Apply changes from delta to gold layers
    pass

# Tasks
bronze_to_delta_task = PythonOperator(
    task_id='bronze_to_delta',
    python_callable=bronze_to_delta,
    dag=dag,
)

delta_to_gold_task = PythonOperator(
    task_id='delta_to_gold',
    python_callable=delta_to_gold,
    dag=dag,
)

# Define the workflow
bronze_to_delta_task >> delta_to_gold_task
```

**Example input / output:**

```text
# Example workflow

Input to bronze layer:
- id=1, data_column='example', timestamp=2023-01-01T00:00:00Z

Processing in delta layer (delta_to_gold):
- INSERT id=1, action='INSERT'
- UPDATE id=1, action='UPDATE' // hypothetical example
- DELETE id=1, action='DELETE'

Output to gold aggregate:
- After processing: aggregated_data_column for id 1 is updated/removed as per delta actions.
```

**Data:**

```json
{
  "bronze": {
    "schema": [
      {"name": "id", "type": "INT"},
      {"name": "data_column", "type": "VARCHAR(255)"},
      {"name": "timestamp", "type": "TIMESTAMP"}
    ]
  },
  "delta": {
    "schema": [
      {"name": "id", "type": "INT"},
      {"name": "action", "type": "VARCHAR(10)"} // 'INSERT', 'UPDATE', 'DELETE'
    ]
  },
  "gold": {
    "schema": [
      {"name": "id", "type": "INT"},
      {"name": "aggregated_data_column", "type": "DECIMAL"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Bronze Layer
4. Delta Layer
5. Gold Aggregate Layer
6. Data Warehouse
7. Business Intelligence/AnalysisTools
8. Is Data Modification Required?
9. Apply Delta Changes to GoldLayer
10. Skip Delta Processing

