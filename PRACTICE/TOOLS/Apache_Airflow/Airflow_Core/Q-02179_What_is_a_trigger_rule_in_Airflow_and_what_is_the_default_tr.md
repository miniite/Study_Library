---
type: "practice"
difficulty: "Intermediate"
---

# What is a trigger rule in Airflow and what is the default trigger rule?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Scheduling Retries and Trigger Rules](<../../../../TOOLS/Apache_Airflow/Airflow_Core/Scheduling_Retries_and_Trigger_Rules.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Airflow, a trigger rule is a mechanism that defines under which conditions a task should be executed if one or more of its upstream tasks fail. The default trigger rule in Airflow is "all_failed," meaning that the downstream task will only be triggered if all its upstream tasks have failed. This section will explore what trigger rules are, the default trigger rule, and how to use them effectively.

TL;DR

A trigger rule in Apache Airflow dictates when a task should run based on the state of its upstream tasks. The default is "all_failed," meaning a downstream task runs only if all upstream tasks have failed. Understanding different trigger rules helps in designing robust workflows that handle failures gracefully.

#### Triage
What's Being Probed

The interviewer might be probing your understanding of Airflow’s internal workflow mechanics, particularly how tasks interact and when they are executed based on their dependencies. You need to explain the default behavior and discuss potential scenarios where changing the trigger rule could be beneficial.

#### First-Principles Solution

In Apache Airflow, a task's execution is triggered by its upstream tasks' states. By default, if an upstream task fails (indicated by a state of "FAILED"), the downstream task will only run if all upstream tasks have failed ("all_failed" trigger rule). This behavior ensures that you do not proceed with processing unless all previous steps have failed, which is particularly useful for critical operations.

Default Trigger Rule: all_failed
Behavior: The downstream task runs only when all its upstream tasks are in a "FAILED" state.
Use Case: Ideal for ensuring that no further processing occurs if any part of the workflow has encountered an error. It helps maintain data integrity and consistency by stopping the execution early.
Other Common Trigger Rules
all\_success: The downstream task runs only when all upstream tasks are successful ("SUCCESS").
one\_failed: The downstream task runs if at least one upstream task is in a "FAILED" state.
one\_succeeded: The downstream task runs if at least one upstream task is in a "SUCCESS" state.
all\_success\_or\_failed: The downstream task runs if all tasks are either successful or failed.
none\_of\_the_above: The downstream task runs regardless of the upstream task states.

#### Implementation
Example

Here's an example of setting up trigger rules in Airflow using Python:

PYTHON
from airflow import DAG
from airflow.utils.trigger_rule import TriggerRule
from datetime import timedelta

default_args = {
    'owner': 'airflow',
    'depends\_on\_past': False,
    'start\_date': days.ago(2),
}

dag = DAG(
    'example_dag',
    default\_args=default\_args,
    description='Example DAG with different trigger rules',
    schedule_interval=timedelta(hours=1),
)

t1 = BashOperator(
    task\_id='task1',
    bash\_command='exit 0',  # Example command
    dag=dag,
)

t2 = BashOperator(
    task\_id='task2',
    bash\_command='exit 1',  # Example command
    trigger\_rule=TriggerRule.ONE\_FAILED,  # Custom trigger rule
    dag=dag,
)

#### Closing Summary
+ Common Mistakes/Follow-Ups

Understanding the default and different trigger rules in Airflow is crucial for designing robust workflows. The all_failed default helps ensure that downstream tasks are not executed unless all upstream tasks have failed, which can be important for maintaining data integrity. However, depending on your use case, you might want to adjust these rules to allow certain tasks to run even if some upstream tasks fail.

#### Common mistakes
include:

Not considering the impact of different trigger rules in complex workflows.
Ignoring the need to handle failures gracefully, leading to potential data inconsistencies.
Failing to test and validate trigger rules thoroughly before deployment.

#### Follow-ups
could involve discussing how to implement custom trigger rules using Python, or exploring scenarios where all\_success might be more appropriate than all\_failed.

**Code:**

```python
from airflow import DAG
from airflow.utils.trigger_rule import TriggerRule
from datetime import timedelta

default_args = {
    'owner': 'airflow',
    'depends_on_past': False,
    'start_date': days.ago(2),
}

dag = DAG(
    'example_dag',
    default_args=default_args,
    description='Example DAG with different trigger rules',
    schedule_interval=timedelta(hours=1),
)

t1 = BashOperator(
    task_id='task1',
    bash_command='exit 0',  # Example command
    dag=dag,
)

t2 = BashOperator(
    task_id='task2',
    bash_command='exit 1',  # Example command
    trigger_rule=TriggerRule.ONE_FAILED,  # Custom trigger rule
    dag=dag,
)
```

