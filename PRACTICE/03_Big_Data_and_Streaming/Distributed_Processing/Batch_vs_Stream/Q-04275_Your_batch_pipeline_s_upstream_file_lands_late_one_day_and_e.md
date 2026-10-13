---
type: "practice"
difficulty: "Deep"
---

# Your batch pipeline's upstream file lands late one day and everything downstream cascades. How do you make the DAG resilient to late-arriving inputs?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Streaming Correctness and Drift](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Batch_vs_Stream/Streaming_Correctness_and_Drift.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a late-arriving input file in your batch processing pipeline, we need to ensure that the Directed Acyclic Graph (DAG) remains robust and minimizes downstream job disruptions. This problem is common in data pipelines where the input files can be delayed due to various reasons such as network issues or upstream systems' latencies.

#### Triage

The core issue here involves making sure your DAG design allows for late-arriving inputs without failing or delaying other dependent tasks. We need to explore concepts like backfilling, retry mechanisms, and handling dependencies in a way that minimizes cascading failures. Key assumptions include:

The pipeline is designed using Airflow or a similar tool.
Late-arriving files are infrequent but could happen due to external factors.

#### First Principles Solution

To make the DAG resilient, we can implement several strategies:

Use Backfill Mechanism: Ensure that late-arriving input files are reprocessed with minimal delay using backfilling techniques.
Improve Input Handling Logic: Modify the input handling logic to accommodate late-arrivals gracefully.
Optimize DAG Dependencies: Design dependencies between tasks to handle late-arrivals effectively.
Step-by-Step Solution

Identify Late-Arriving Jobs:

Use Airflow's DependsOnPast flag or similar mechanism to identify and re-run dependent tasks when a critical input is late.

Backfill Strategy:

Implement backfill logic in the DAG definition using parameters like start_date, end_date, and retries.

#### Example
Define a task with a dynamic start date that can be updated to include late-arriving data.

Late-Arrival Handling:

Modify the code to handle late-arrivals by checking for files in both the current and past time periods.
Use conditional logic to only process new or updated files, avoiding redundant work.

Dependency Management:

Ensure tasks are designed with appropriate dependencies that can adapt to missing inputs by using task_instance.run to manually trigger downstream tasks.

Logging and Monitoring:

Implement logging to track which files were processed and when.
Set up monitoring to alert on late-arriving data or failing tasks.

#### Closing Summary

By implementing a backfill strategy, improving input handling logic, managing dependencies effectively, and enhancing monitoring, we can ensure that our DAG remains resilient to late-arrivals. This approach not only minimizes disruptions but also provides insights into potential issues within the pipeline.

#### Common Mistakes/Follow-Ups
Mistake: Failing to account for backfilling in critical tasks.

#### Follow-Up
Discuss how to balance the cost of reprocessing with the benefits of timely data availability.
Mistake: Ignoring late-arriving files, leading to incomplete or incorrect downstream results.

#### Follow-Up
Explore solutions like data versioning and incremental processing to handle partial data sets.

**Comparison:**

```
No
Yes
Start Date
Is File Path Confirmed?
Fail Task and Alert
Process Data
Log Processed Data
Maintain Dependency on PastJobs
Monitor for Late Arrivals
```

**Code:**

```python
# Example Airflow DAG with backfill logic

from datetime import timedelta
from airflow import DAG
from airflow.utils.dates import days_ago
from airflow.operators.python_operator import PythonOperator

default_args = {
    'owner': 'data_engineer',
    'depends_on_past': True,
    'start_date': days_ago(2),
}

dag = DAG(
    'late_arriving_input_dag',
    default_args=default_args,
    description='Example DAG for handling late-arriving inputs',
    schedule_interval=timedelta(days=1),
)

def process_data(**kwargs):
    # Logic to process data and handle late arrivals
    file_path = kwargs['dag_run'].conf.get('file_path')
    if not file_path:
        return

    with open(file_path, 'r') as f:
        # Process the data here
        pass

process_task = PythonOperator(
    task_id='process_data',
    python_callable=process_data,
    provide_context=True,
    dag=dag,
)
```

**Example input / output:**

| Task ID | Input File Path        | Start Date       | End Date      |
|---------|-----------------------|------------------|---------------|
| process_data | /data/inputs/input_20231001.csv | 2023-10-01T00:00:00Z | 2023-10-05T00:00:00Z |

**Data:**

```json
{
  "file_path": "/data/inputs/input_20231001.csv",
  "start_date": "2023-10-01",
  "end_date": "2023-10-05"
}
```

**Flow:**

1. Yes
2. No
3. Detect Late Arrival
4. Backfill Previous Dates
5. Is Data Processed?
6. Continue Processing OtherJobs
7. Retry Task or Alert

