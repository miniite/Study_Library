---
type: "practice"
difficulty: "Deep"
---

# How would you enforce and evolve a data contract between an upstream producer team and your feature pipeline?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Schema Drift Handling](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Schema_Drift_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce and evolve a data contract in a data pipeline, we need to ensure that the data produced by the upstream producer team aligns with the expectations of our feature pipeline. This involves both static validation at runtime and dynamic contract management as requirements change over time. The key steps include defining the contract schema, implementing robust checks, and maintaining an evolving strategy for contract enforcement.

To begin, we need to clearly define what constitutes a valid data contract. For instance, if the producer team provides CSV files with specific columns like customer_id, transaction_amount, and timestamp, then these should always be present in the incoming data stream. This can be defined using schemas such as Avro or JSON Schema.

Next, we'll integrate this schema into our data processing pipeline to enforce it during runtime. For a more detailed approach, let's consider an example where we use Apache Airflow for workflow management and Apache Spark for data processing.

**Code:**

```python
from airflow.models import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

# Define the DAG with a clear task ID, schedule interval, etc.
dag = DAG(
    'data_pipeline',
    schedule_interval='@daily',
    start_date=datetime(2023, 1, 1),
)

# Define the Spark operator to enforce data contract
def check_data_contract(data):
    # Implement validation logic here
    if not all(key in data for key in ['customer_id', 'transaction_amount', 'timestamp']):
        raise ValueError("Missing required fields")

check_operator = SparkSubmitOperator(
    task_id='validate_data',
    application='/path/to/validation_script.py',
    dag=dag,
)
```

**Data:**

```json
{
  "fields": [
    {"name": "customer_id", "type": "int"},
    {"name": "transaction_amount", "type": "float"},
    {"name": "timestamp", "type": "string"}
  ]
}
```

