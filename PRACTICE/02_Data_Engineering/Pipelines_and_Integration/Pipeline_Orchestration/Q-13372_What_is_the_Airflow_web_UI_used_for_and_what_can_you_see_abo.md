---
type: "practice"
difficulty: "Basic"
---

# What is the Airflow web UI used for and what can you see about a DAG there?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Airflow DAGs and Backfills](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Airflow_DAGs_and_Backfills.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The Airflow Web UI is an essential tool for managing, monitoring, and debugging Data Processing Pipelines (DAGs). It provides a visual interface to view the status of tasks, logs, and dependencies within a DAG. By interacting with the Web UI, you can gain insights into the health and performance of your workflows.

The Airflow Web UI allows you to see several key pieces of information about a DAG:

#### Overview
This section gives an overall view of the DAG, including its scheduled intervals, last successful execution time, and current state.
Graph View: Here, you can visualize the dependencies between tasks in your DAG as a directed acyclic graph (DAG). This helps you understand how tasks are interconnected and the flow of data through your pipeline.
Task Instances: You can see individual task instances for each run of the DAG. Each instance is color-coded to show its state, such as running, success, or failure.
Logs: The Web UI provides access to logs for each task instance, which are crucial for diagnosing issues and understanding the execution flow.
Backfill: This feature allows you to manually run historical data through your DAG, useful for backfilling missed runs or testing changes.

Let's dive into the first-principles solution step-by-step:

Understanding Airflow Components:

Airflow is an open-source platform designed to programmatically author, schedule, and monitor workflows.
A Directed Acyclic Graph (DAG) represents a workflow as a collection of tasks. Each task can have dependencies on other tasks.

Role of the Web UI:

The Web UI serves as a user interface for interacting with Airflow's backend. It is accessible via a web browser and provides a high-level overview of the DAGs and their execution.

#### Overview
Section:

This section shows key metrics such as the number of tasks, the scheduled interval (CRON expression), and the last run status.

Graph View:

The graph view displays the dependencies between tasks. Each node represents a task, and arrows indicate the direction of data flow.
You can hover over nodes to see more details about each task's state, such as whether it is running or failed, and why.

Task Instances:

Clicking on a DAG in the overview section takes you to the Task Instances page, where you can see detailed information for each run.
You can filter by date ranges, states (success, failure, etc.), and even drill down into specific task instances.

Logs:

The Web UI provides access to logs directly from within the browser, making it easier to troubleshoot issues without needing to manually query the database or log files.
You can filter logs by date, task instance ID, or even search for specific keywords.

Backfilling:

Using the Web UI's backfill feature, you can run historical data through your DAGs, which is particularly useful for testing changes in a safe manner without affecting live systems.
This feature helps ensure that all data points are processed correctly and can be used to verify the correctness of new code.

In summary, the Airflow Web UI is a powerful tool for managing and monitoring Data Pipelines. It provides an intuitive interface to view, debug, and manage DAGs, making it easier to maintain complex workflows.

**Comparison:**

```
Task 1
Task 2
```

**Code:**

```python
# Example Python script using Airflow's backfill feature
from airflow import DAG
from airflow.utils.dates import days_ago

dag = DAG(
    'example_dag',
    start_date=days_ago(1),
    schedule_interval="@daily",
)

with dag:
    task_1 = BashOperator(task_id='task_1', bash_command="echo 'Task 1'")
    task_2 = PythonOperator(task_id='task_2', python_callable=lambda: print("Task 2"))

# To backfill the DAG, run:
# airflow dags backfill -s YYYY-MM-DD -e YYYY-MM-DD example_dag
```

**Example input / output:**

```text
# Example of Task Instances view in Airflow Web UI

| Task Instance ID | State | Start Date         | End Date           |
|------------------|-------|--------------------|--------------------|
| ti_1             | Success | 2023-10-02T10:00:00Z | 2023-10-02T10:05:00Z |
| ti_2             | Failed  | 2023-10-02T10:10:00Z | 2023-10-02T10:15:00Z |

Logs for Task Instance ID 'ti_2':
```

**Data:**

```yaml
example-dag.yaml
---
dag_id: example_dag
schedule_interval: '@daily'
start_date: 2023-10-01
tasks:
  task_1:
    type: BashOperator
    command: echo 'Task 1'
  task_2:
    type: PythonOperator
    callable: print("Task 2")
```

**Flow:**

1. Start
2. DAG Loaded
3. Determine Schedule Interval
4. Task Instances Scheduled
5. Execute Task Instances
6. Check Task Completion
7. Update State in Metadata DB
8. End
9. Backfill Requested? Yes -->J[Run Backfill]

