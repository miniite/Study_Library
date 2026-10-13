---
type: "practice"
difficulty: "Basic"
---

# How do task dependencies work in an Airflow DAG, and what does writing extract >> transform >> load in Python express?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Idempotent Airflow Tasks](<../../../../TOOLS/Python/Python_Language/Idempotent_Airflow_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how task dependencies work in an Airflow DAG using Python, let's start by briefly summarizing what we're dealing with. An Airflow DAG (Directed Acyclic Graph) is a way to define workflows as a collection of tasks that depend on each other. In Python, these tasks are typically defined within the default_args and with DAG() context managers.

Now, let's dive into the concept of task dependencies in an Airflow DAG and what "extract >> transform >> load" (ETL) means in this context.

#### Triaging the Question

The question is asking about how task dependencies are handled in Airflow using Python. Specifically, it references ETL tasks, which are common in data processing pipelines. The >> operator in an Airflow DAG is used to define these dependencies. It tells Airflow that one task must complete before another can start.

#### First Principles Solution

To understand this fully, let's break down the key components and steps:

Defining a Task: In Python within an Airflow DAG, we create tasks using functions decorated with @task or @dag_task.
Specifying Dependencies: We use the >> operator to define when one task should run after another.
DAG Context Manager: The entire DAG is defined inside a context manager provided by the Airflow library.
Step-by-Step Solution

Creating Tasks:

PYTHON
from airflow.decorators import task, dag

@task
def extract():
    # Code to extract data goes here
    return "extracted_data"

@task
def transform(data):
    # Code to transform the extracted data goes here
    return f"transformed_{data}"

@task
def load(transformed_data):
    # Code to load the transformed data goes here
    return "loaded_data"

Defining Dependencies:

PYTHON
from datetime import timedelta

@dag(
    default_args={
        'owner': 'airflow',
        'retries': 1,
        'retry_delay': timedelta(minutes=5),
    },
    schedule_interval=timedelta(days=1),
    start_date=datetime(2023, 1, 1),
    catchup=False
)
def etl_dag():
    extracted_data = extract()

    transformed_data = transform(extracted_data) >> load(transformed_data)

dag = etl_dag()

#### Closing Summary
and Common Mistakes/Follow-Ups

In the example above, extract is a task that runs first. Once it completes successfully, its output is passed to transform, which then passes its transformed data to load. The key here is using the >> operator to create these dependencies.

#### Common mistakes
include:

Not defining tasks properly (e.g., not importing necessary decorators).
Forgetting to pass the correct arguments between tasks.
Misunderstanding the >> syntax and not realizing it defines a sequence of operations.

#### Follow-ups
could involve discussing how Airflow handles task failures, retry mechanisms, or more advanced dependency patterns like parallel execution.

**Code:**

```

from airflow.decorators import task, dag

@task def extract(): # Code to extract data goes here return "extracted_data"

@task def transform(data): # Code to transform the extracted data goes here return f"transformed_{data}"

@task def load(transformed_data): # Code to load the transformed data goes here return "loaded_data"

CODE
Copy

```python
from datetime import timedelta

@dag(
    default_args={
        'owner': 'airflow',
        'retries': 1,
        'retry_delay': timedelta(minutes=5),
    },
    schedule_interval=timedelta(days=1),
    start_date=datetime(2023, 1, 1),
    catchup=False
)
def etl_dag():
    extracted_data = extract()

    transformed_data = transform(extracted_data) >> load(transformed_data)

dag = etl_dag()
```

**Flow:**

1. Extract
2. Transform
3. Load

