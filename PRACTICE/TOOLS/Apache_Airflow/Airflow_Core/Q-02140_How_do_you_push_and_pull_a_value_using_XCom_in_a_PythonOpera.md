---
type: "practice"
difficulty: "Intermediate"
---

# How do you push and pull a value using XCom in a PythonOperator task?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Airflow Architecture](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Airflow_Architecture.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how to push and pull values using XCom in an Apache Airflow PythonOperator task, let’s start by briefly summarizing what we’re dealing with. XCom (pronounced "ex-com") is a mechanism in Apache Airflow that allows tasks within the same or different DAGs to communicate with each other by storing and retrieving data between them. In this case, we’ll focus on using PythonOperator tasks.

#### Triaging the Question

The question is asking for an explanation of how to pass values (push) from one task to another in a PythonOperator. Specifically, it’s looking at XCom as a means of achieving this communication between tasks. The core idea here is understanding the Python API provided by Airflow and how to leverage it within your DAGs.

#### First Principles Solution

To push and pull values using XCom in a PythonOperator, you need to understand two main concepts: pushing data into an XCom from one task, and then pulling that data from another task. We'll break this down step-by-step:

Pushing Data (From Task A): Use the ti.xcom_push method within your Python function.
Pulling Data (In Task B): Use the ti.xcom_pull method to retrieve the pushed data.

#### Example
Code

Here’s a simple example of how you might implement this in Airflow:

PYTHON
# Import necessary modules from Airflow
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime

def push_value(**context):
    # Pushing a value using XCom
    context['ti'].xcom_push(key='key_name', value='some_value')

def pull_value(**context):
    # Pulling the value pushed by the previous task
    pulled_value = context['ti'].xcom_pull(task_ids='task_a', key='key_name')
    print(f"Pulled Value: {pulled_value}")

# Initialize the DAG
dag = DAG(
    'example_xcom_dag',
    start_date=datetime(2023, 1, 1),
    schedule_interval=None,
)

# Define tasks
task_a = PythonOperator(
    task_id='task_a',
    python_callable=push_value,
    dag=dag,
)

task_b = PythonOperator(
    task_id='task_b',
    python_callable=pull_value,
    dag=dag,
)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, XCom allows you to communicate between tasks in Airflow by pushing and pulling values using the xcom_push and xcom_pull methods. A common mistake is forgetting to import the necessary modules or not correctly referencing the task instance (ti) when calling these methods.

For a more advanced use case, consider managing multiple key-value pairs, handling data serialization/deserialization, and understanding how XCom behaves with dependencies between tasks.

If asked follow-up questions, you might discuss:

How Airflow handles race conditions with XCom.
The impact of XCom on the performance of your DAGs.
Alternative ways to achieve task communication in Airflow, such as using sensors or events.

**Code:**

```python
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime

def push_value(**context):
    context['ti'].xcom_push(key='key_name', value='some_value')

def pull_value(**context):
    pulled_value = context['ti'].xcom_pull(task_ids='task_a', key='key_name')
    print(f"Pulled Value: {pulled_value}")

dag = DAG(
    'example_xcom_dag',
    start_date=datetime(2023, 1, 1),
    schedule_interval=None,
)

task_a = PythonOperator(
    task_id='task_a',
    python_callable=push_value,
    dag=dag,
)

task_b = PythonOperator(
    task_id='task_b',
    python_callable=pull_value,
    dag=dag,
)
```

**Example input / output:**

```

Example input -> output:

PLAINTEXT
Copy
Input:
- Task A executes `push_value` and pushes 'some_value' with key 'key_name'.

Output:
- Task B executes `pull_value`, retrieves the pushed value, and prints "Pulled Value: some_value".
```

**Flow:**

1. push('key_name','some_value')
2. Task A
3. Context['ti'].xcom_pull(task_ids='task_a', key='key_name')
4. Prints 'Pulled Value:some_value'

