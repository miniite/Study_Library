---
type: "practice"
difficulty: "Deep"
---

# How do you design a DAG so that a partial failure leaves the warehouse in a consistent state — think transactional boundaries and compensation for a multi-step load?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ DAG Design and Dependencies](<../../../../TOOLS/Apache_Airflow/Airflow_Core/DAG_Design_and_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure consistency in the warehouse during a partial failure, we need to design an Apache Airflow DAG that handles transactions with atomicity. The key idea is to use transactional boundaries and compensation strategies to handle failures gracefully without leaving the data in an inconsistent state. This involves dividing the load process into smaller tasks, ensuring each step has its own transactional boundary, and implementing rollback mechanisms if any part of the process fails.

**Code:**

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime

def load_data_step1(**kwargs):
    # Step 1 Data Loading logic
    return "Step 1 Success"

def load_data_step2(**kwargs):
    # Step 2 Data Loading logic
    return "Step 2 Success"

def load_data_step3(**kwargs):
    # Step 3 Data Loading logic
    raise Exception("Simulated failure")

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG('consistent_load_dag', default_args=default_args, schedule_interval='@daily')

with dag:
    step1 = PythonOperator(
        task_id='load_step1',
        python_callable=load_data_step1,
        provide_context=True
    )

    step2 = PythonOperator(
        task_id='load_step2',
        python_callable=load_data_step2,
        provide_context=True
    )

    step3 = PythonOperator(
        task_id='load_step3',
        python_callable=load_data_step3,
        provide_context=True
    )

    step1 >> step2 >> step3  # Assuming a linear dependency for simplicity
```

**Example input / output:**

```text
No specific input-output scenarios required, but note that in practice:
- `load_data_step1` would return "Step 1 Success"
- `load_data_step2` might proceed to the next step if successful
- `load_data_step3` failure should be caught and handled appropriately
```

**Flow:**

1. Yes
2. Yes
3. No
4. Load Step 1
5. Step 2 Success?
6. Load Step 2
7. Step 3 Success?
8. Load Step 3
9. Compensate Step 2 & Step 3

