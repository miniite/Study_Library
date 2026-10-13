---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between Airflow's LocalExecutor and SequentialExecutor for a beginner setup?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Airflow DAGs and Backfills](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Airflow_DAGs_and_Backfills.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When setting up data pipelines, understanding the differences between executors in Apache Airflow can be crucial for choosing the right one based on your use case. For beginners, it's important to grasp that Airflow uses executors to run tasks across workers. The LocalExecutor and SequentialExecutor are two simple executors used in basic setups.

The key difference lies in how these executors manage task execution:

TL;DR: LocalExecutor runs all tasks locally on the same machine, while SequentialExecutor runs tasks sequentially within a single worker process.

**Comparison:**

```
Executor Type	Task Execution Model
LocalExecutor	Runs all tasks locally
SequentialExecutor	Runs tasks sequentially within a single worker process
```

**Code:**

```
Example DAG definition with LocalExecutor

from airflow import DAG from airflow.operators.python_operator import PythonOperator

def task_function(): print("Executing local task")

with DAG(dag_id='local_executor_dag', start_date=datetime(2023, 1, 1)) as dag: t1 = PythonOperator( task_id='local_task', python_callable=task_function, executor='LocalExecutor' )

CODE
Copy

```python
# Example DAG definition with SequentialExecutor
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

def task_function():
    print("Executing sequential task")

with DAG(dag_id='sequential_executor_dag', start_date=datetime(2023, 1, 1)) as dag:
    t1 = PythonOperator(
        task_id='sequential_task',
        python_callable=task_function,
        executor='SequentialExecutor'
    )
```

**Flow:**

1. LocalExecutor
2. Run tasks in parallel
3. All tasks run on the samemachine
4. SequentialExecutor
5. Run tasks sequentially
6. Task order is preserved withina single process

