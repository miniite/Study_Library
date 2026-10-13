---
type: "practice"
difficulty: "Intermediate"
---

# How would you handle an API that occasionally times out so your extract task doesn't fail permanently?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Non-critical Task Failures](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a timeout during the extraction phase in our data pipeline, we need to ensure that our system can gracefully handle such failures without causing permanent disruptions. The key idea here is to implement retry logic with appropriate backoff and error handling mechanisms.

This question is essentially probing how you would design fault-tolerant systems, which are crucial for robust data pipelines. It involves understanding the nature of timeouts, managing retries effectively, and ensuring that your solution scales well without overwhelming the system.

#### Triaging the Question

The core of the problem revolves around handling intermittent API timeouts in an extract task using a reliable framework like Apache Airflow or Spark. We need to consider:

Understanding the source: Why does the API occasionally time out?
Retry mechanism: How many times should we retry, and with what delay?
Error tracking: How can we log and monitor these failures?

#### First-Principles Solution

To solve this problem, let's break it down step by step:

Understand the Timeout Mechanism:

API timeouts are often due to network issues, high load on the server, or temporary service unavailability.

Implement Retry Logic in Airflow/Spark:

Use a retry mechanism that retries the task up to a certain number of times before failing permanently.

Backoff Strategy:

Implement an exponential backoff strategy to avoid overwhelming the API with too many requests in quick succession.

Error Tracking and Logging:

Log detailed information about each failure, including timestamp, error message, and any relevant metadata.

Notification Mechanisms:

Set up notifications for critical failures so that stakeholders are alerted if a task fails repeatedly.
Step-by-Step Solution

Let’s consider how we would implement this in an Airflow DAG using Python:

PYTHON
from datetime import timedelta
from airflow import DAG
from airflow.providers.http.sensors.http import HttpSensor
from airflow.providers.http.operators.http import SimpleHttpOperator

default_args = {
    'owner': 'data_engineer',
    'retries': 3,
    'retry_delay': timedelta(minutes=2),
}

dag = DAG(
    'api_timeout_example',
    default_args=default_args,
    description='A simple example to handle API timeouts in Airflow',
    schedule_interval=timedelta(days=1),
)

def failure_callback(context):
    """Send alert for critical failures."""
    import smtplib
    from email.mime.text import MIMEText

    # SMTP setup and sending an email
    msg = MIMEText(f"Task {context['task_instance'].task_id} failed.")
    msg["Subject"] = "API Extraction Failure"
    msg["From"] = "data_engineer@example.com"
    msg["To"] = "alert@example.com"

    with smtplib.SMTP("smtp.example.com", 587) as server:
        server.sendmail(msg["From"], [msg["To"]], msg.as_string())

# Create an HttpSensor to check if the API is available
check_api_availability = HttpSensor(
    task_id='check_api_availability',
    http_conn_id='api_connection',
    endpoint='/api/status',
    timeout=60,  # Allow a bit more time than usual for this initial check
    poke_interval=30,
    retries=2,  # Retry once if the API is not available initially
)

# Use an HttpOperator to fetch data from the API with retry mechanism
extract_data = SimpleHttpOperator(
    task_id='extract_data',
    http_conn_id='api_connection',
    endpoint='/api/data',  # Endpoint for data extraction
    method='GET',
    retries=3,  # Retry up to three times if there is a timeout or failure
    on_failure_callback=failure_callback,
)

check_api_availability >> extract_data

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary
In summary, handling API timeouts effectively involves implementing robust retry mechanisms with appropriate backoff strategies. Additionally, setting up proper error tracking and notification systems ensures that the failure can be managed in a timely manner.

#### Common Mistakes

Over-relying on exponential backoff without proper limits.
Not configuring retries properly, leading to potential infinite loops or too many failed attempts.
Ignoring logging and notifications, which are crucial for debugging and monitoring.

#### Follow-Ups
How would you ensure the robustness of your retry logic in a distributed system?
If you had a choice between using Airflow or Spark for this task, which one would you choose and why?

**Code:**

```python
from datetime import timedelta

default_args = {
    'owner': 'data_engineer',
    'retries': 3,
    'retry_delay': timedelta(minutes=2),
}

dag = DAG(
    'api_timeout_example',
    default_args=default_args,
    description='A simple example to handle API timeouts in Airflow',
    schedule_interval=timedelta(days=1),
)

def failure_callback(context):
    import smtplib
    from email.mime.text import MIMEText

    msg = MIMEText(f"Task {context['task_instance'].task_id} failed.")
    msg["Subject"] = "API Extraction Failure"
    msg["From"] = "data_engineer@example.com"
    msg["To"] = "alert@example.com"

    with smtplib.SMTP("smtp.example.com", 587) as server:
        server.sendmail(msg["From"], [msg["To"]], msg.as_string())

check_api_availability = HttpSensor(
    task_id='check_api_availability',
    http_conn_id='api_connection',
    endpoint='/api/status',
    timeout=60,
    poke_interval=30,
    retries=2,
)

extract_data = SimpleHttpOperator(
    task_id='extract_data',
    http_conn_id='api_connection',
    endpoint='/api/data',
    method='GET',
    retries=3,
    on_failure_callback=failure_callback,
)

check_api_availability >> extract_data
```

