---
type: "practice"
difficulty: "Basic"
---

# What is Apache Airflow, and what does a DAG represent when you author a data pipeline as a Python file?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Idempotent Airflow Tasks](<../../../../TOOLS/Python/Python_Language/Idempotent_Airflow_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Apache Airflow is an open-source platform to programmatically author, schedule, and monitor workflows. When using Airflow with Python, you define these workflows in Directed Acyclic Graphs (DAGs). A DAG represents a sequence of tasks that are logically dependent on each other, forming a directed acyclic graph. This means there are no cycles or loops; tasks can only depend on previous tasks.

**Comparison:**

```
Task ID	Task Type
print_hello	PythonOperator
```

**Code:**

```python
from airflow import DAG
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'retries': 1,
}

dag = DAG(
    dag_id='example_dag',
    start_date=datetime(2023, 7, 1),
    default_args=default_args,
    schedule_interval="@daily"
)
```

**Example input / output:**

```text
# Example task in the DAG
from airflow.operators.python_operator import PythonOperator

def print_hello():
    return 'Hello, World!'

hello_task = PythonOperator(
    task_id='print_hello',
    python_callable=print_hello,
    dag=dag
)
```

**Flow:**

1. Yes
2. Start Date
3. Is Today's Date >= Start Date?
4. DAG Activated
5. Task 1 (print_hello)
6. Task 2 (Other Tasks)
7. End of DAG

