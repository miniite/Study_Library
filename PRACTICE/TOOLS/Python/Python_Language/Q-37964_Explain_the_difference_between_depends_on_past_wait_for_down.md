---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between depends_on_past, wait_for_downstream, and trigger rules in Airflow, and how you'd use them to enforce ordering in a Python data pipeline.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Airflow Tasks](<../../../../TOOLS/Python/Python_Language/Idempotent_Airflow_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Apache Airflow, managing dependencies and triggering rules is crucial for ensuring that tasks execute in the correct order and under specific conditions. For a Data Engineer at an intermediate level like yours, understanding depends_on_past, wait_for_downstream, and trigger rules will help you design robust pipelines that can handle complex workflows efficiently.

To break it down:

Depends on Past: This flag is used to specify whether the task should only run if its previous execution was successful.
Wait for Downstream: This ensures that a downstream task waits for all upstream tasks to complete before starting.
Trigger Rules: These are more complex and allow you to define custom conditions under which a task should be executed.

Now, let's dive into how these work and when you might use them in a Python data pipeline.

TL;DR
depends_on_past: Ensures the current execution depends on the success of its previous execution.
wait_for_downstream: Ensures downstream tasks wait for upstream tasks to complete before starting.
Trigger rules: Define custom conditions that determine when a task should be executed, providing flexibility beyond simple dependency checks.

#### Triage

The question asks you to explain these concepts and how they enforce ordering in a Python data pipeline. The key points are:

Depends on Past: Checks if the previous run of the task was successful.
Wait for Downstream: Ensures tasks are ordered correctly without waiting for dependencies within the same DAG (Directed Acyclic Graph).
Trigger Rules: Offers more complex logic to control when a task should be executed.

#### First-Principles Solution

Let's start with depends_on_past:

PYTHON
from airflow.models import DAG
from datetime import timedelta

dag = DAG(
    'example_dag',
    default_args={'owner': 'airflow', 'retries': 1, 'retry_delay': timedelta(minutes=5)},
    depends_on_past=True  # This flag ensures the current execution only runs if the previous one was successful.
)

Next, wait_for_downstream:

PYTHON
from airflow.models import DAG

dag = DAG(
    'example_dag',
    default_args={'owner': 'airflow', 'retries': 1},
    wait_for_downstream=True  # This ensures all upstream tasks are completed before this task starts.
)

Finally, trigger rules:

PYTHON
from airflow.models import BaseOperator
from airflow.utils.trigger_rule import TriggerRule

class CustomOperator(BaseOperator):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)

    def execute(self, context):
        # Custom logic here
        pass

CustomOperator(
    task_id='custom_task',
    trigger_rule=TriggerRule.ONE_SUCCESS  # This ensures the task runs if at least one upstream task succeeds.
)

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary:

depends_on_past is used to ensure a task's current run depends on its previous successful execution.
wait_for_downstream ensures tasks are ordered by waiting for all upstream tasks to complete before starting, but it doesn’t depend on the success of each individual upstream task.
Trigger rules provide more complex logic and flexibility in determining when tasks should be executed.

#### Common mistakes
include:

Misunderstanding the difference between depends_on_past and wait_for_downstream.
Not considering edge cases where a single upstream failure could impact downstream tasks differently based on the chosen trigger rule.
Overlooking the need for robust error handling and retries when dealing with complex dependencies.

#### Follow-up
questions might include:

How do these concepts interact in nested DAGs?
What are best practices for using these features together?

