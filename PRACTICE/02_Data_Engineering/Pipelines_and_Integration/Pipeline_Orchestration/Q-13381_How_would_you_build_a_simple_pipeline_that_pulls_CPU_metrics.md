---
type: "practice"
difficulty: "Intermediate"
---

# How would you build a simple pipeline that pulls CPU metrics every hour and stores them for later anomaly detection?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline Data Monitoring](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Pipeline_Data_Monitoring.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a simple data pipeline that pulls CPU metrics every hour and stores them for later anomaly detection, we'll use Apache Airflow to orchestrate the tasks and Apache Spark to process and store the data. Here’s a high-level overview of how this would work:

#### Overview
We will create an Airflow DAG (Directed Acyclic Graph) that runs a Spark task every hour to fetch CPU metrics from a source, transform them if necessary, and then store them in a database or file system for further analysis.

#### Triage
The interviewer is likely probing your understanding of:

How Airflow works and its configuration.
Your knowledge of Spark and how it integrates with Airflow.
Best practices for data storage and retrieval.
Error handling and logging mechanisms in the pipeline.

#### First-Principles Solution

Airflow Setup: We need to set up an Airflow environment, define a DAG that triggers hourly, and write a Spark job within this DAG.
Spark Task: The Spark task will interact with a database or API to fetch CPU metrics, process them (e.g., calculate averages), and store the results in a structured format.

Detailed Steps:

Define the Airflow Environment: Install Airflow and necessary dependencies like apache-airflow, pyspark, etc.

Create the DAG File: Define an Airflow DAG with a schedule interval of '1 hour'.

PYTHON
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2023, 5, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'cpu_metrics_pipeline',
    default_args=default_args,
    description='A simple pipeline to pull and store CPU metrics every hour.',
    schedule_interval=timedelta(hours=1)
)

Python Operator for Spark Task: Create a Python function that uses Spark to fetch, process, and store the data.

PYTHON
import pyspark
from pyspark.sql.functions import avg

def pull_cpu_metrics():
    spark = pyspark.sql.SparkSession.builder.getOrCreate()
    # Fetch CPU metrics (example: from a database or API)
    cpu_metrics_df = fetch_cpu_data_from_source()  # Placeholder function
    # Process the data (e.g., calculate average CPU usage per hour)
    avg_cpu_usage = cpu_metrics_df.groupBy('timestamp').agg(avg('cpu_percent').alias('avg_cpu'))
    # Store processed data in a structured format
    avg_cpu_usage.write.parquet('/path/to/output')

Define the PythonOperator: Register the Spark task as an Airflow operator.

PYTHON
from airflow.operators.python_operator import PythonOperator

pull_cpu_metrics_task = PythonOperator(
    task_id='pull_cpu_metrics',
    python_callable=pull_cpu_metrics,
    dag=dag,
)

Error Handling and Logging: Implement proper logging to capture any issues during data fetching, processing, or storage.

PYTHON
from airflow.utils.log.logging_mixin import LoggingMixin

class CustomLoggingOperator(PythonOperator):
    def __init__(self, *args, **kwargs):
        super(CustomLoggingOperator, self).__init__(*args, **kwargs)

    def execute(self, context):
        log = LoggingMixin().log
        try:
            return super(CustomLoggingOperator, self).execute(context)
        except Exception as e:
            log.error(f"Error occurred: {e}")
            raise

pull_cpu_metrics_task = CustomLoggingOperator(
    task_id='pull_cpu_metrics',
    python_callable=pull_cpu_metrics,
    dag=dag,
)

#### Closing Summary
By setting up an Airflow DAG that triggers a Spark task every hour, we can efficiently fetch and store CPU metrics for later analysis. Proper logging ensures that any issues are captured and can be addressed.

#### Common Mistakes/Follow-Ups

Mistake: Not considering the data source properly (e.g., ensuring it's accessible and reliable).

#### Follow-Up
Discussing how to handle intermittent failures in fetching metrics.
Mistake: Ignoring performance considerations when writing large datasets using Spark.

#### Follow-Up
Explaining strategies for optimizing Spark jobs, such as partitioning and caching.

**Comparison:**

```
Timestamp	CPU Percent
2023-10-01T00:00:00Z	85
2023-10-01T01:00:00Z	90
2023-10-01T02:00:00Z	70
2023-10-01T03:00:00Z	60
2023-10-01T04:00:00Z	45

Processed Output:

Timestamp	Avg CPU Usage
2023-10-01T00:00:00Z	82.6
2023-10-01T01:00:00Z	85
2023-10-01T02:00:00Z	75
2023-10-01T03:00:00Z	62.4
2023-10-01T04:00:00Z	55
```

**Code:**

```python
from datetime import timedelta
from airflow import DAG
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': datetime(2023, 5, 1),
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
}

dag = DAG(
    'cpu_metrics_pipeline',
    default_args=default_args,
    description='A simple pipeline to pull and store CPU metrics every hour.',
    schedule_interval=timedelta(hours=1)
)

def fetch_cpu_data_from_source():
    # Placeholder function for fetching data
    pass

class CustomLoggingOperator(PythonOperator):
    def __init__(self, *args, **kwargs):
        super(CustomLoggingOperator, self).__init__(*args, **kwargs)

    def execute(self, context):
        log = LoggingMixin().log
        try:
            return super(CustomLoggingOperator, self).execute(context)
        except Exception as e:
            log.error(f"Error occurred: {e}")
            raise

def pull_cpu_metrics():
    spark = pyspark.sql.SparkSession.builder.getOrCreate()
    cpu_metrics_df = fetch_cpu_data_from_source()  # Placeholder function
    avg_cpu_usage = cpu_metrics_df.groupBy('timestamp').agg(avg('cpu_percent').alias('avg_cpu'))
    avg_cpu_usage.write.parquet('/path/to/output')

pull_cpu_metrics_task = CustomLoggingOperator(
    task_id='pull_cpu_metrics',
    python_callable=pull_cpu_metrics,
    dag=dag,
)
```

**Example input / output:**

```text
Input:
- CPU metrics from a source API or database.

Output:
- A structured parquet file containing the average CPU usage per hour.
```

**Data:**

```json
{
  "cpu_percent": [85, 90, 70, 60, 45],
  "timestamp": ["2023-10-01T00:00:00Z", "2023-10-01T01:00:00Z", "2023-10-01T02:00:00Z", "2023-10-01T03:00:00Z", "2023-10-01T04:00:00Z"]
}
```

**Flow:**

1. Start
2. Define Airflow DAG
3. Create Spark Task
4. Implement Data FetchingLogic
5. Process and Store Data
6. Log Errors and Failures
7. End

