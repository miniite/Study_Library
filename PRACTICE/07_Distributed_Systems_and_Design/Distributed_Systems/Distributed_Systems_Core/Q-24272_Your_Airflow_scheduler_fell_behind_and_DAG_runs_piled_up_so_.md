---
type: "practice"
difficulty: "Deep"
---

# Your Airflow scheduler fell behind and DAG runs piled up so a downstream task ran on data an upstream hadn't produced yet. Explain how scheduler distribution executor choice and sensors/dependencies prevent this out-of-order execution.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that downstream tasks in Apache Airflow do not execute on outdated or incomplete data, we need to carefully manage the scheduling and task dependencies within our system. The key components here are the scheduler's distribution strategy, the choice of executors, and the effective use of sensors and dependencies.

**Comparison:**

```
Task ID	Dependencies	Execution Logic
produce_data	None	Simulates data production.
wait_for_new_data	None	Waits for new data in the database based on a SQL query.
process_data	[produce_data, wait_for_new_data]	Processes the latest available data only after both upstream tasks complete successfully.
```

**Code:**

```python
# Example DAG with a sensor and dependency setup
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow.sensors.sql_sensor import SqlSensor

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG(
    'example_dag',
    default_args=default_args,
    schedule_interval=timedelta(hours=1)
)

def produce_data():
    # Simulate data production
    pass

produce_task = PythonOperator(
    task_id='produce_data',
    python_callable=produce_data,
    dag=dag
)

def process_data():
    # Simulate data processing using the latest produced data
    pass

process_task = PythonOperator(
    task_id='process_data',
    python_callable=process_data,
    dag=dag
)

# Ensure process_data runs only after produce_data completes
produce_task >> process_task

# Use a sensor to wait for new data production before proceeding
new_data_sensor = SqlSensor(
    task_id='wait_for_new_data',
    conn_id='your_database_connection_id',
    sql="SELECT COUNT(*) FROM your_table WHERE produced_at > (SELECT MAX(produced_at) FROM your_table)",
    poke_interval=30,
    timeout=1800,
    dag=dag
)

new_data_sensor >> process_task
```

**Example input / output:**

```text
Input:
- `produce_data()` function triggers at every schedule interval.
- The produced data is stored in the database with a timestamp indicating when it was created.

Output:
- After a new piece of data is produced, the sensor checks for any new entries with a more recent `produced_at` timestamp than the latest existing entry.
- If such data exists, the `process_data()` task runs; otherwise, the process waits until new data becomes available.
```

**Data:**

```json
{
  "database_connection_id": "your_database_connection_id",
  "table_name": "your_table",
  "produced_at_column": "produced_at"
}
```

**Flow:**

1. Produce Process Group
2. Produce Data
3. Process Data
4. Wait for New Data

