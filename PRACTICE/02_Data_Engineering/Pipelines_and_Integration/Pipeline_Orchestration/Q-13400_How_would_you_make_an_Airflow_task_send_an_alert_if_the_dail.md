---
type: "practice"
difficulty: "Intermediate"
---

# How would you make an Airflow task send an alert if the daily metrics pipeline fails?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Non-critical Task Failures](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the problem of making an Airflow task send an alert when a daily metrics pipeline fails, we need to understand the typical structure and components of Airflow tasks and how they interact with external systems like email or Slack for sending alerts.

Firstly, let's break down what the question is asking. The core issue is ensuring that if a specific task in the daily metrics pipeline encounters an error, a notification should be sent out. This involves setting up conditional logic within your Airflow DAG (Directed Acyclic Graph) to detect failures and trigger alerting mechanisms.

#### Assumptions

We are using Airflow 2.x.
The failing task is part of a DAG that runs daily.
Alerts can be sent via email or Slack.

We will start by creating a simple Airflow DAG and then add the necessary logic for handling task failure events. Here’s how we would approach it:

Define the DAG Structure: Create a DAG with a daily schedule interval and include the failing task within this structure.
Set Up Task Dependencies: Ensure that if one task fails, the subsequent tasks are aware of this error.
Implement Error Handling in Tasks: Use Airflow’s on_failure_callback to define what happens when a task fails.
Configure Alerting Mechanisms: Set up an alerting mechanism within the task failure callback.

Now, let's walk through these steps:

Define the DAG Structure

PYTHON
from datetime import timedelta
from airflow import DAG
from airflow.providers.slack.operators.slack_webhook import SlackWebhookOperator

default_args = {
    'owner': 'airflow',
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    'on_failure_callback': failure_callback_function, # Define this function below
}

dag = DAG(
    'daily_metrics_pipeline',
    default_args=default_args,
    description='A daily metrics pipeline',
    schedule_interval='@daily',
    start_date=datetime(2023, 1, 1),
    catchup=False,
)

Set Up Task Dependencies We will define a task that processes the daily metrics and add it to our DAG.

PYTHON
from airflow.decorators import task

@task(task_id='process_daily_metrics')
def process_data():
    # Your data processing logic here
    pass

process_data_task = process_data()

Implement Error Handling in Tasks We will use the on_failure_callback to handle task failures.

PYTHON
from airflow.exceptions import AirflowSensorTimeout, TaskDeferred

def failure_callback_function(context):
    slack_webhook_token = 'YOUR_SLACK_WEBHOOK_TOKEN'
    message = f"Task {context['task_instance'].task_id} failed with an error: \n{context['exception']}"
    SlackWebhookOperator(
        task_id='send_alert',
        http_conn_id=None,
        webhook_token=slack_webhook_token,
        message=message
    ).execute(context)

process_data_task.on_failure_callback = failure_callback_function

Configure Alerting Mechanisms The failure_callback_function defined above is responsible for sending the alert via Slack. This function will be triggered whenever process_data fails.

#### Closing Summary
By following these steps, we ensure that if the task process_daily_metrics fails, an alert is sent out through a Slack notification. The key elements here are defining the DAG structure, setting up the task dependencies, and implementing the error handling logic using Airflow’s built-in features like on_failure_callback. This approach ensures robust monitoring of the daily metrics pipeline.

#### Common Mistakes

Not properly setting up the on_failure_callback for tasks.
Failing to include appropriate error messages in alerts.
Neglecting to test the alert mechanism thoroughly.

