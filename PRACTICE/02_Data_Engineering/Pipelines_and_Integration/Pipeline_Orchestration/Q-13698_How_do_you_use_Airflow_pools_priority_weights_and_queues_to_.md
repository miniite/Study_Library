---
type: "practice"
difficulty: "Deep"
---

# How do you use Airflow pools priority weights and queues to guarantee the daily critical pipeline preempts an ad-hoc backfill on a shared cluster?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Orchestration vs Processing](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that your daily critical data pipeline preempts an ad-hoc backfill task in Airflow, you need to effectively use pools and priority weights in conjunction with queues. This solution involves configuring Airflow to prioritize the execution of tasks based on their importance, while also managing resource allocation across different workflows.

Firstly, let’s break down what we are trying to achieve:

Pipelines: We have a daily critical pipeline that needs to run every day.
Backfill: An ad-hoc backfill task that may need to be run at any time and must not interfere with the daily pipeline execution.

To accomplish this, we will use Airflow’s concept of pools and queues:

#### Triaging the Question

Airflow’s pools feature allows you to group tasks and limit the number of concurrent tasks within a pool. Priority weights, on the other hand, allow you to set relative priorities for tasks in different queues. By carefully configuring these, we can ensure that our critical pipeline gets executed first.

Step-by-Step Solution
1. Define Pools

Define two pools: one for your daily critical pipeline and another for ad-hoc backfills.

PYTHON
# In Airflow's `airflow.cfg`
[webserver]
default_pool = default

[pool dailycritical]
slots = 2

[pool adhocbackfill]
slots = 4
2. Create Queues

Create two queues: one for your daily pipeline and another for backfills.

PYTHON
# In Airflow's `airflow.cfg`
[queues]
default = default
dailycritical = dailycritical
adhocbackfill = adhocbackfill
3. Assign Tasks to Queues and Pools

Assign tasks to the appropriate queues and pools in your DAGs.

PYTHON
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from airflow.utils.task_group import TaskGroup

default_args = {
    'owner': 'dataengineer',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'email': ['dataengineer@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
}

dag = DAG(
    'critical_pipeline_dag',
    default_args=default_args,
    description='Daily critical pipeline with ad-hoc backfill preemption',
    schedule_interval=timedelta(days=1),
)

with TaskGroup('daily_critical_tasks', tooltip="Daily critical tasks") as daily_critical_group:
    critical_task_1 = SparkSubmitOperator(
        task_id='critical_task_1',
        application='/path/to/critical_script.py',
        queue='dailycritical',
        pool='dailycritical',
        priority_weight=5,
        dag=dag
    )

    critical_task_2 = SparkSubmitOperator(
        task_id='critical_task_2',
        application='/path/to/another_critical_script.py',
        queue='dailycritical',
        pool='dailycritical',
        priority_weight=4,
        dag=dag
    )

with TaskGroup('adhoc_backfill_tasks', tooltip="Ad-hoc backfill tasks") as adhoc_backfill_group:
    backfill_task_1 = SparkSubmitOperator(
        task_id='backfill_task_1',
        application='/path/to/backfill_script.py',
        queue='adhocbackfill',
        pool='adhocbackfill',
        priority_weight=3,
        dag=dag
    )

    backfill_task_2 = SparkSubmitOperator(
        task_id='backfill_task_2',
        application='/path/to/another_backfill_script.py',
        queue='adhocbackfill',
        pool='adhocbackfill',
        priority_weight=2,
        dag=dag
    )

# Set dependencies to ensure critical tasks are run first
critical_task_1 >> critical_task_2 >> adhoc_backfill_group
4. Configure Priority Weights and Queues

The priority_weight setting ensures that higher-priority tasks in the dailycritical queue preempt lower-priority tasks in the adhocbackfill queue if both queues are running on the same cluster.

The critical pipeline has a priority weight of 5, making it higher than the backfill task's weights (3 and 2).

#### Closing Summary

By using Airflow’s pools and queues effectively, we can ensure that our daily critical pipeline preempts any ad-hoc backfills. This setup prioritizes the execution of important tasks while managing resource allocation efficiently.

#### Common Mistakes/Follow-Ups
Incorrect Pool Configuration: Make sure the pool settings match your cluster's capacity.
Overloading Queues: Ensure that the number of slots in each queue is sufficient to handle both daily and ad-hoc tasks without causing bottlenecks.
Priority Weights vs. Task Dependencies: Be cautious about setting priority weights; they should complement task dependencies, not contradict them.

By following this approach, you can ensure robust data pipeline management in a shared cluster environment.

**Comparison:**

| Task ID          | Application Path             | Queue         | Pool       | Priority Weight |
|------------------|------------------------------|---------------|------------|-----------------|
| critical_task_1  | /path/to/critical_script.py   | dailycritical | dailycritical | 5               |
| critical_task_2  | /path/to/another_critical_script.py | dailycritical | dailycritical | 4               |
| backfill_task_1  | /path/to/backfill_script.py   | adhocbackfill | adhocbackfill  | 3               |
| backfill_task_2  | /path/to/another_backfill_script.py | adhocbackfill | adhocbackfill  | 2               |

**Code:**

```python
from airflow import DAG
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator
from airflow.utils.task_group import TaskGroup

default_args = {
    'owner': 'dataengineer',
    'depends_on_past': False,
    'start_date': datetime(2023, 1, 1),
    'email': ['dataengineer@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
}

dag = DAG(
    'critical_pipeline_dag',
    default_args=default_args,
    description='Daily critical pipeline with ad-hoc backfill preemption',
    schedule_interval=timedelta(days=1),
)

with TaskGroup('daily_critical_tasks', tooltip="Daily critical tasks") as daily_critical_group:
    critical_task_1 = SparkSubmitOperator(
        task_id='critical_task_1',
        application='/path/to/critical_script.py',
        queue='dailycritical',
        pool='dailycritical',
        priority_weight=5,
        dag=dag
    )

    critical_task_2 = SparkSubmitOperator(
        task_id='critical_task_2',
        application='/path/to/another_critical_script.py',
        queue='dailycritical',
        pool='dailycritical',
        priority_weight=4,
        dag=dag
    )

with TaskGroup('adhoc_backfill_tasks', tooltip="Ad-hoc backfill tasks") as adhoc_backfill_group:
    backfill_task_1 = SparkSubmitOperator(
        task_id='backfill_task_1',
        application='/path/to/backfill_script.py',
        queue='adhocbackfill',
        pool='adhocbackfill',
        priority_weight=3,
        dag=dag
    )

    backfill_task_2 = SparkSubmitOperator(
        task_id='backfill_task_2',
        application='/path/to/another_backfill_script.py',
        queue='adhocbackfill',
        pool='adhocbackfill',
        priority_weight=2,
        dag=dag
    )

# Set dependencies to ensure critical tasks are run first
critical_task_1 >> critical_task_2 >> adhoc_backfill_group
```

**Example input / output:**

```text
# Example input to airflow.cfg
[webserver]
default_pool = default

[pool dailycritical]
slots = 2

[pool adhocbackfill]
slots = 4

[queues]
default = default
dailycritical = dailycritical
adhocbackfill = adhocbackfill
```

**Data:**

```plaintext
[webserver]
default_pool = default

[pool dailycritical]
slots = 2

[pool adhocbackfill]
slots = 4

[queues]
default = default
dailycritical = dailycritical
adhocbackfill = adhocbackfill
```

**Flow:**

1. DAG Definition
2. Daily Critical Tasks
3. Critical Task 1 (Priority Weight5)
4. Critical Task 2 (Priority Weight4)
5. Adhoc Backfill Tasks
6. Backfill Task 1 (Priority Weight3)
7. Backfill Task 2 (Priority Weight2)

