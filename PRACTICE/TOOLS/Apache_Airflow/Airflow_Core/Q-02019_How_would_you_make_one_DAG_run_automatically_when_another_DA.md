---
type: "practice"
difficulty: "Deep"
---

# How would you make one DAG run automatically when another DAG finishes producing a dataset?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DAG Design and Dependencies](<../../../../TOOLS/Apache_Airflow/Airflow_Core/DAG_Design_and_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that one Apache Airflow Directed Acyclic Graph (DAG) runs automatically when another DAG produces a dataset, we can use dependencies and the sla_miss mechanism in Airflow. This approach ensures that the dependent DAG starts only after the producer DAG has completed successfully or failed with an SLA miss.

#### Triage

The core question is about setting up a dependency relationship between two DAGs where one should trigger based on the completion of another. Key points to consider include:

Ensuring the producer DAG completes before the consumer DAG starts.
Using sla_miss to handle cases where the producer DAG doesn't complete within its scheduled interval.

#### First Principles Solution

We can achieve this by configuring a dependency in the dependent DAG and using SLA Miss notifications for robustness. Here's a step-by-step breakdown:

Producer DAG Configuration:

Define the tasks that produce the dataset.
Set up an sla (Service Level Agreement) on the producer DAG to specify how long it should take to complete.

#### Cons
umer DAG Configuration:

Identify the task in the consumer DAG that depends on the output from the producer DAG.
Use the depends_on_past parameter or a custom trigger rule if needed.
Optionally, use an sla_miss to handle delays gracefully.

#### Example
Implementation

Here's how you can implement this:

Producer DAG (ProducerDag.py)
PYTHON
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': '2023-01-01',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    'sla': timedelta(hours=1),  # SLA set for 1 hour
}

dag = DAG(
    dag_id='producer_dag',
    default_args=default_args,
    schedule_interval='@daily',
)

def produce_data():
    print("Producing data...")

produce_task = PythonOperator(
    task_id='produce_data',
    python_callable=produce_data,
    dag=dag,
)

#### Cons
umer DAG (ConsumerDag.py)
PYTHON
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'airflow',
    'depends_on_past': True,  # This ensures the consumer depends on the producer's completion
    'start_date': '2023-01-01',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    dag_id='consumer_dag',
    default_args=default_args,
    schedule_interval='@daily',
)

def consume_data():
    print("Consuming data...")

#### cons
ume_task = PythonOperator(
    task_id='consume_data',
    python_callable=consume_data,
    dag=dag,
)
SLA Miss Handling

To handle cases where the producer DAG fails to complete within its SLA, you can set up an sla_miss callback:

PYTHON
def sla_callback(context):
    """
    This function is called when a task in the producer DAG misses its SLA.
    You can implement custom logic here, such as sending notifications or logging errors.
    """
    print(f"Producer DAG missed its SLA. Notifying team...")

from airflow.models import TaskInstance
producer_dag = 'producer_dag'

#### cons
umer_dag = 'consumer_dag'

sla_miss_callback = sla_callback

# Schedule the consumer task to depend on the producer's completion and SLA miss

#### cons
ume_task.set_downstream(produce_task)

#### cons
ume_task.sla_miss_callback = sla_miss_callback

#### Closing Summary
+ Common Mistakes/Follow-Ups

This setup ensures that consumer_dag runs only after producer_dag completes successfully or misses its SLA. Key points to remember:

Use depends_on_past to ensure the consumer depends on the producer.
Configure an appropriate sla for the producer DAG.
Implement sla_miss_callback for robust error handling.

#### Common mistakes
include not setting the correct dependencies, failing to understand the nuances of Airflow's SLA mechanism, and not considering edge cases where the producer might fail or be delayed.

Would you like any further clarification on this setup?

**Comparison:**

| DAG        | Schedule Interval | Producer Task  | Consumer Task |
|------------|-------------------|----------------|---------------|
| producer_dag | @daily            | produce_data   | -             |
| consumer_dag | @daily           | consume_data   | depends on producer_dag and sla_miss_callback |

**Code:**

```python
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

# ProducerDag.py
default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': '2023-01-01',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    'sla': timedelta(hours=1),  # SLA set for 1 hour
}

dag = DAG(
    dag_id='producer_dag',
    default_args=default_args,
    schedule_interval='@daily',
)

def produce_data():
    print("Producing data...")

produce_task = PythonOperator(
    task_id='produce_data',
    python_callable=produce_data,
    dag=dag,
)

# ConsumerDag.py
default_args = {
    'owner': 'airflow',
    'depends_on_past': True,  # This ensures the consumer depends on the producer's completion
    'start_date': '2023-01-01',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    dag_id='consumer_dag',
    default_args=default_args,
    schedule_interval='@daily',
)

def consume_data():
    print("Consuming data...")

consume_task = PythonOperator(
    task_id='consume_data',
    python_callable=consume_data,
    dag=dag,
)

# SLA Miss Handling
from airflow.models import TaskInstance
producer_dag = 'producer_dag'
consumer_dag = 'consumer_dag'

def sla_callback(context):
    """
    This function is called when a task in the producer DAG misses its SLA.
    You can implement custom logic here, such as sending notifications or logging errors.
    """
    print(f"Producer DAG missed its SLA. Notifying team...")

# Schedule the consumer task to depend on the producer's completion and SLA miss
consume_task.set_downstream(produce_task)
consume_task.sla_miss_callback = sla_callback
```

**Example input / output:**

```text
Input:
ProducerDag.py and ConsumerDag.py are defined as shown in the code blocks.

Output:
When `producer_dag` completes within its SLA, `consumer_dag` starts. If `producer_dag` misses its SLA, a callback is triggered to handle the delay.
```

**Data:**

```json
{
  "producer_dag": {
    "dag_id": "producer_dag",
    "default_args": {
      "owner": "airflow",
      "depends_on_past": false,
      "start_date": "2023-01-01",
      "retries": 1,
      "retry_delay": "5 minutes",
      "sla": "1 hour"
    },
    "schedule_interval": "@daily"
  },
  "consumer_dag": {
    "dag_id": "consumer_dag",
    "default_args": {
      "owner": "airflow",
      "depends_on_past": true,
      "start_date": "2023-01-01",
      "retries": 1,
      "retry_delay": "5 minutes"
    },
    "schedule_interval": "@daily"
  }
}
```

**Flow:**

1. @daily
2. On success or SLA Miss
3. producer_dag
4. consumer_dag
5. produce_data
6. consume_data

