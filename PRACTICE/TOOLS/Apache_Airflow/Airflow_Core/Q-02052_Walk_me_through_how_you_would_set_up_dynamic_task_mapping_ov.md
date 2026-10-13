---
type: "practice"
difficulty: "Deep"
---

# Walk me through how you would set up dynamic task mapping over a variable list of files and cap concurrency so you do not overwhelm a downstream API.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Airflow Architecture](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Airflow_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To set up dynamic task mapping over a variable list of files in Apache Airflow while capping concurrency, we need to leverage Airflow's flexibility with tasks, dynamically generating tasks based on file lists and setting limits to avoid overwhelming downstream APIs. The key concepts involve using Python operators, custom hooks, and configuration parameters.

**Comparison:**

| Task ID | File Processed |
|---------|----------------|
| process_0 | file1.txt     |
| process_1 | file2.txt     |
| process_2 | file3.txt     |

**Code:**

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from airflow.utils.task_group import TaskGroup

def process_file(file_path):
    # Function that processes a single file
    pass

def dynamic_task_generator(dag, files_list):
    task_dict = {}
    for i, file in enumerate(files_list):
        task_id = f"process_{i}"
        task = PythonOperator(
            task_id=task_id,
            python_callable=process_file,
            op_args=[file],
            dag=dag
        )
        task_dict[task_id] = task

    return task_dict

with DAG(dag_id='dynamic_files_dag', schedule_interval=None, start_date=datetime(2023, 1, 1)) as dag:
    with TaskGroup(group_id='process_files_group') as process_files_group:
        files_list = get_file_list()  # Assume this function fetches the list of files
        tasks = dynamic_task_generator(dag, files_list)

    # Setting concurrency limits for all tasks in the group
    max_concurrency = 5
    for task in tasks.values():
        task.override.priority_weight = max_concurrency

    # Example downstream API call
    call_downstream_api()
```

**Example input / output:**

```text
Input:
- List of files to process: ['file1.txt', 'file2.txt', 'file3.txt']

Output:
- Tasks generated and executed with the specified concurrency limit.
  - Task "process_0" processes file1.txt.
  - Task "process_1" processes file2.txt.
  - Task "process_2" processes file3.txt.

Concurrency Limit Applied:
- Each task is assigned a priority weight of 5, ensuring no more than 5 tasks run concurrently.
```

**Data:**

```python
def get_file_list():
    return ['file1.txt', 'file2.txt', 'file3.txt']
```

**Flow:**

1. Dynamic Files DAG
2. Get File List
3. Generate Tasks
4. Set Priority Weight for EachTask
5. Run Tasks Concurrency Limited
6. Call Downstream API

