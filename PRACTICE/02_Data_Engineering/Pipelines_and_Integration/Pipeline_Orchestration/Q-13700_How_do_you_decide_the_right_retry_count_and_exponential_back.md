---
type: "practice"
difficulty: "Intermediate"
---

# How do you decide the right retry count and exponential backoff for a task calling a flaky third-party API without masking a real persistent failure?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Handling Non-critical Task Failures](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To determine the appropriate retry count and exponential backoff strategy for tasks in a data pipeline that interact with a potentially unreliable third-party API, we need to balance reliability and efficiency. The goal is to handle transient failures without overwhelming our system or masking actual persistent issues.

#### Triage

The core of this question revolves around understanding how to design fault tolerance mechanisms effectively in your data pipeline. Specifically, the interviewer is probing into your understanding of:

Transient vs Persistent Failures: Distinguishing between temporary network glitches and permanent issues.
Retry Mechanisms: How many times should a task retry before considering it failed?
Exponential Backoff: The strategy to increase wait times after each retry to avoid overwhelming the system or the third-party API.

#### Assumptions
We are using Airflow for orchestrating our data pipeline.
The third-party API may experience intermittent issues but not always due to permanent errors.
There is a risk of masking persistent issues with retries, which could lead to incorrect processing results.

#### First-Principles Solution

Identify Transient Failures:

Temporary network failures are common and should be handled by retrying the task. However, we need to ensure that such temporary failures do not mask actual permanent issues.

Determine Retry Count:

Typically, a small number of retries (e.g., 3-5) is sufficient for handling transient errors. This can be configured in Airflow using the retries parameter.

Exponential Backoff Strategy:

Exponential backoff helps manage retries by increasing the wait time between attempts to avoid overwhelming the system or the third-party API.
A common approach is to use a base delay and multiply it exponentially for each subsequent retry. For example, starting with 1 second, then 2 seconds, 4 seconds, etc.

#### Example
Configuration

In Airflow, you can set up these parameters in your DAG definition:

PYTHON
from datetime import timedelta
from airflow.models import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'retries': 3,
    'retry_delay': timedelta(seconds=1),
}

dag = DAG(
    dag_id='third_party_api_dag',
    default_args=default_args,
    schedule_interval='@daily',
)

def call_third_party_api():
    # Your API calling logic here
    pass

call_api_task = PythonOperator(
    task_id='call_third_party_api_task',
    python_callable=call_third_party_api,
    retries=3,  # Number of retries
    retry_delay=timedelta(seconds=1),  # Initial delay for the first retry
)

#### Closing Summary
and Common Mistakes
Mistake: Setting too many retries (e.g., >5) can lead to resource exhaustion if transient errors persist. Conversely, setting a very low number might not adequately handle intermittent issues.
Mistake: Not implementing exponential backoff can result in overwhelming the third-party API or your system with repeated requests.
Best Practice: Regularly monitor the failure rate and adjust retry configurations as needed.
Common Follow-Ups
How do you detect persistent failures and handle them differently from transient ones?
What metrics would you collect to measure the effectiveness of your retry strategy?
Can you discuss how this approach scales with increasing API load?

**Code:**

```python
from datetime import timedelta
from airflow.models import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'retries': 3,
    'retry_delay': timedelta(seconds=1),
}

dag = DAG(
    dag_id='third_party_api_dag',
    default_args=default_args,
    schedule_interval='@daily',
)

def call_third_party_api():
    # Your API calling logic here
    pass

call_api_task = PythonOperator(
    task_id='call_third_party_api_task',
    python_callable=call_third_party_api,
    retries=3,  # Number of retries
    retry_delay=timedelta(seconds=1),  # Initial delay for the first retry
)
```

**Data:**

```json
{
  "dag": {
    "dag_id": "third_party_api_dag",
    "default_args": {
      "retries": 3,
      "retry_delay": "0:01:00"
    },
    "schedule_interval": "@daily",
    "tasks": [
      {
        "task_id": "call_third_party_api_task",
        "python_callable": "call_third_party_api",
        "retries": 3,
        "retry_delay": "0:00:01"
      }
    ]
  }
}
```

