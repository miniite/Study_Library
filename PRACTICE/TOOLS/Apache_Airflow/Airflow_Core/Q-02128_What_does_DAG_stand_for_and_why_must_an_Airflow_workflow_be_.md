---
type: "practice"
difficulty: "Basic"
---

# What does DAG stand for and why must an Airflow workflow be acyclic?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DAG Design and Dependencies](<../../../../TOOLS/Apache_Airflow/Airflow_Core/DAG_Design_and_Dependencies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: A Directed Acyclic Graph (DAG) in Apache Airflow represents a collection of tasks that need to be executed, where each task is represented as a node and dependencies between tasks are represented by directed edges. The DAG ensures that the workflow is acyclic because it prevents circular references or loops, which could lead to infinite loops during execution.

#### Triaging the Question
The question is asking about two aspects of Apache Airflow:

What does "DAG" stand for in the context of Airflow?
Why must an Airflow workflow be acyclic?

#### Assumptions

We are dealing with version 1.x or later, as DAGs were introduced to manage dependencies and workflows.
The interviewer is interested in understanding the fundamental concepts underlying Airflow's workflow management system.

Step-by-Step First Principles Solution: A Directed Acyclic Graph (DAG) is a data structure used in Apache Airflow for defining workflows. Here’s how it works:

Nodes: Each node represents a task or an operation.
Edges: Directed edges represent dependencies between tasks, indicating the order of execution.
Acyclicity Requirement: The graph must be acyclic to ensure that there are no circular references.

The reason for the acyclicity requirement is rooted in preventing infinite loops and ensuring predictable and manageable workflows:

Task Ordering: Tasks with dependencies (edges) can only start once their prerequisites (dependencies) have completed. This ensures a clear order of execution.
Avoiding Loops: A cycle would mean a task depends on itself, leading to an infinite loop where the task never completes or enters into an undefined state.

In practice, consider a simple example:

CODE
Task 1 -> Task 2

Here, Task 1 must complete before Task 2 starts. If we had Task 3 also depending on Task 2, and Task 2 depended back on Task 1, it would create a cycle.

#### Closing Summary
+ Common Mistakes/Follow-Ups: Understanding DAGs is crucial for effective use of Airflow as they form the backbone of how tasks are orchestrated. Key mistakes to avoid include:

Overlooking dependency cycles, which can lead to workflow issues.
Not defining dependencies clearly enough, resulting in ambiguous task relationships.

For follow-ups, one could explore additional aspects such as:

How DAGs handle retries and failures.
The role of start_date, end_date, and schedule_interval in managing tasks over time.
Strategies for optimizing complex workflows using multi-DAG structures or parallelism.

**Comparison:**

| Task  | Dependencies |
|-------|--------------|
| Task 1| None         |
| Task 2| Task 1       |

**Code:**

```python
# Example of a simple DAG definition in Airflow
from airflow import DAG
from datetime import datetime

default_args = {
    'owner': 'airflow',
    'start_date': datetime(2023, 10, 1),
}

dag = DAG(
    'example_dag',
    default_args=default_args,
    schedule_interval='@daily',  # This means the dag runs daily
)

def task_1():
    print("Task 1 started")

task_1_task = PythonOperator(task_id='task_1', python_callable=task_1, dag=dag)

def task_2():
    print("Task 2 started")

task_2_task = PythonOperator(task_id='task_2', python_callable=task_2, dag=dag)

# Task 2 depends on Task 1
task_2_task >> task_1_task
```

**Example input / output:**

```text
Input:
- Task 1 runs and prints "Task 1 started"
- Task 2 depends on Task 1

Output:
1. "Task 1 started" is printed.
2. After Task 1 completes, "Task 2 started" is printed.

Dry-run scenario with a cycle (to illustrate the error):
- Attempting to create a cycle between Task A and Task B: Task A -> Task B -> Task A
- This would result in Airflow throwing an error due to detection of a cyclic dependency.
```

**Data:**

```json
{
    "dag": {
        "id": "example_dag",
        "default_args": {
            "owner": "airflow",
            "start_date": "2023-10-01"
        },
        "schedule_interval": "@daily",
        "tasks": [
            {"task_id": "task_1", "dependencies": []},
            {"task_id": "task_2", "dependencies": ["task_1"]}
        ]
    }
}
```

**Flow:**

1. Task 1
2. Task 2

