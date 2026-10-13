---
type: "practice"
difficulty: "Deep"
---

# How would you set up retries and SLAs in Airflow so a transient Prometheus outage doesn't cascade into a missed daily anomaly-baseline refresh?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Handling Non-critical Task Failures](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to ensure that our data pipeline setup using Apache Airflow is resilient against temporary outages in Prometheus while still maintaining SLAs for critical tasks like refreshing anomaly baselines. The key idea here is to implement robust retry mechanisms and define Service Level Agreements (SLAs) that account for potential delays.

First, let's break down the requirements:

Transient Outage Handling: We need Airflow to handle temporary outages in Prometheus without failing the task.
SLA Compliance: The pipeline should meet its deadlines even if there are transient issues, which can be achieved by setting appropriate retry policies and SLAs.

To achieve this, we'll leverage Airflow’s built-in features such as retry_on_failure and sla_miss_callback, and use best practices in task design. Here's a step-by-step approach:

Task Design: Define the tasks involved in the pipeline, specifically focusing on the Prometheus-based data processing and anomaly baseline refresh.
Retry Mechanism: Implement retry logic to ensure that if a transient outage occurs, the task will attempt to run again without failing immediately.
SLA Configuration: Configure SLAs to define acceptable latencies for critical tasks.

Now, let's dive into the details:

Step 1: Task Design

We'll create an Airflow DAG (Directed Acyclic Graph) that processes data from Prometheus and refreshes anomaly baselines. The key task in this pipeline is refresh_anomaly_baseline, which depends on the availability of recent data from Prometheus.

PYTHON
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from datetime import timedelta

default_args = {
    'owner': 'airflow',
    'retries': 3,  # Number of retries for transient errors
    'retry_delay': timedelta(minutes=5),  # Delay between retries
}

with DAG(
    dag_id='prometheus_data_pipeline',
    default_args=default_args,
    schedule_interval='@daily',  # Run daily at midnight
    start_date=datetime(2023, 1, 1),
    catchup=False,
) as dag:

    refresh_anomaly_baseline = SparkSubmitOperator(
        task_id='refresh_anomaly_baseline',
        application='/path/to/anomaly_baseline_refresh.py',
        conn_id='spark_conn_id',
        retry_on_failure=True,  # Ensure the task retries on failure
        sla=timedelta(hours=1),  # SLA to complete within an hour
    )
Step 2: Retry Mechanism

The retry_on_failure attribute ensures that if a transient outage occurs (e.g., Prometheus is temporarily unavailable), Airflow will retry the task. The retry_delay parameter controls how long to wait before the next retry.

Step 3: SLA Configuration

We set an SLA for the refresh_anomaly_baseline task to ensure it completes within one hour of its scheduled time. If the task fails to meet this SLA, Airflow will trigger a callback function that can notify relevant stakeholders or take corrective actions.

PYTHON
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from datetime import timedelta

default_args = {
    'owner': 'airflow',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

with DAG(
    dag_id='prometheus_data_pipeline',
    default_args=default_args,
    schedule_interval='@daily',
    start_date=datetime(2023, 1, 1),
    catchup=False,
) as dag:

    refresh_anomaly_baseline = SparkSubmitOperator(
        task_id='refresh_anomaly_baseline',
        application='/path/to/anomaly_baseline_refresh.py',
        conn_id='spark_conn_id',
        retry_on_failure=True,
        sla=timedelta(hours=1),
    )

#### Common Mistakes
and Follow-Ups
Over-reliance on Retries: While retries are useful, excessive retries can mask underlying issues. Ensure that the tasks being retried have proper logging and error handling to diagnose root causes.
Ignoring SLAs: Failing to meet SLAs can lead to missed deadlines and inaccurate anomaly detection. Regularly monitor SLA compliance and adjust retry policies as needed.
Resource Constraints: Ensure that the resources allocated to the task (e.g., Spark cluster) are sufficient to handle retries without causing overloads.

In conclusion, setting up Airflow for handling transient outages in Prometheus involves a combination of robust retry mechanisms and strict SLA definitions. This setup ensures that critical tasks like anomaly baseline refreshes remain on track even during temporary service disruptions.

**Comparison:**

| Task              | Retries | Retry Delay (min) | SLA (hours) |
|-------------------|---------|------------------|-------------|
| refresh_anomaly_baseline | 3       | 5                | 1           |

**Code:**

```python
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from datetime import timedelta

default_args = {
    'owner': 'airflow',
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
}

with DAG(
    dag_id='prometheus_data_pipeline',
    default_args=default_args,
    schedule_interval='@daily',
    start_date=datetime(2023, 1, 1),
    catchup=False,
) as dag:

    refresh_anomaly_baseline = SparkSubmitOperator(
        task_id='refresh_anomaly_baseline',
        application='/path/to/anomaly_baseline_refresh.py',
        conn_id='spark_conn_id',
        retry_on_failure=True,
        sla=timedelta(hours=1),
    )
```

**Example input / output:**

```text
Input:
- Prometheus outage lasts for 10 minutes
- Scheduled time for refresh_anomaly_baseline is 2 AM

Output:
- Task 'refresh_anomaly_baseline' starts at 2 AM and encounters a Prometheus outage.
- The task retries once after 5 minutes, and again once more after another 5 minutes.
- After the third retry (30 minutes later), the task completes successfully within its SLA window of 1 hour.

Dry-run:
Task 'refresh_anomaly_baseline' -> Prometheus outage -> Retry 1 @ 2:05 AM -> Retry 2 @ 2:10 AM -> Success @ 2:30 AM
```

**Data:**

```json
{
  "dag": {
    "dag_id": "prometheus_data_pipeline",
    "schedule_interval": "@daily",
    "start_date": "2023-01-01"
  },
  "tasks": [
    {
      "task_id": "refresh_anomaly_baseline",
      "operator": "SparkSubmitOperator",
      "application": "/path/to/anomaly_baseline_refresh.py",
      "conn_id": "spark_conn_id",
      "retry_on_failure": true,
      "sla": "1 hour"
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Prometheus Outage
4. Is Prometheus Available?
5. Start Task'refresh_anomaly_baseline'
6. Retry After 5 Minutes
7. Task Retries Again AfterAnother 5 Minutes
8. Task Completes SuccessfullyWithin SLA Window of 1 Hour

