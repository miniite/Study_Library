---
type: "practice"
difficulty: "Deep"
---

# How would you design a canary rollout for a changed dbt transformation so it runs on a sampled schema before promoting to production models?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Drift Handling](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Schema_Drift_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a canary rollout for a modified dbt transformation, we need to ensure that the changes are thoroughly vetted in a controlled environment before being deployed to production. This process involves setting up a sampling strategy to test the new transformations on a subset of data before fully promoting them. Here's how I would approach this problem:

Summary: The key idea is to implement a canary rollout using Airflow and Spark, where we run the new dbt transformation on a sampled schema and validate its output against historical data or expected results.

Restate the Question & Assumptions:

We are working with an existing dbt project that has transformations scheduled in Airflow.
The goal is to introduce changes in a controlled manner, specifically focusing on running these transformations on a sampled schema.
The assumption is that we have access to historical data or baseline results to compare the transformed output.

#### First-Principles Solution

Sampling Strategy: Define a sampling percentage (e.g., 10%) of the full dataset that will be used for testing.
Airflow Task Configuration: Modify the Airflow task responsible for running dbt transformations to use the sampled schema.
Validation Mechanism: Set up a validation step in Airflow to compare the transformed output against historical data or expected results.
Monitoring & Logging: Implement monitoring and logging to track the performance and any issues that arise during the canary rollout.

Detailed Steps:

Define Sampling Percentage: Decide on the sampling percentage based on the size of the dataset. For example, if we have 10 million rows, a 10% sample would be 1 million rows.
Airflow Task Configuration: Update the Airflow task to use the sampled schema. This can be done by modifying the dbt configuration in the Airflow DAG file.
PYTHON
from datetime import datetime

default_args = {
    'owner': 'data_engineer',
    'start_date': datetime(2023, 1, 1),
}

with DAG(
    'canary_rollout_dag',
    default_args=default_args,
    schedule_interval='@daily',
    catchup=False,
) as dag:
    dbt_task = DbtTask(
        task_id='run_canary_transformations',
        profile='your_db_profile',
        project_dir='/path/to/dbt/project',
        select=['+model_name'],
        schema='sampled_schema',
        do_run_tests=True,
        retries=3,
        retry_delay=timedelta(minutes=5),
    )
Validation Mechanism: After the transformation is run, use a validation task to compare the output with historical data or expected results. This can be done using a simple SQL comparison.
PYTHON
validation_task = PythonOperator(
    task_id='validate_output',
    python_callable=validate_output,
    op_kwargs={
        'expected_results_path': '/path/to/expected/results.csv',
        'output_path': '/path/to/output/directory',
    },
)
Monitoring & Logging: Implement monitoring to ensure that the transformations are running smoothly and logging any issues. Use tools like Prometheus or Grafana for real-time monitoring.
PYTHON
monitoring_task = BashOperator(
    task_id='monitor_transformations',
    bash_command="echo 'Monitoring transformations...'",
    trigger_rule=TriggerRule.ALL_SUCCESS,
)

#### Closing Summary
By implementing a canary rollout with Airflow and Spark, we ensure that the new dbt transformations are thoroughly tested before being fully promoted to production. This process helps in catching any issues early and reducing the risk of introducing bugs into the production environment.

#### Common Mistakes/Follow-Ups

#### Common mistakes
include not defining a proper sampling strategy or skipping validation steps, which can lead to undetected errors.

#### Follow-ups
might involve discussing how to automate the validation process, integrating more advanced monitoring tools, and considering additional safeguards like rolling back changes if issues are detected.

**Code:**

```python
from datetime import datetime
from airflow.operators.dbt_operator import DbtTask
from airflow.utils.task_group import TaskGroup

default_args = {
    'owner': 'data_engineer',
    'start_date': datetime(2023, 1, 1),
}

with DAG(
    'canary_rollout_dag',
    default_args=default_args,
    schedule_interval='@daily',
    catchup=False,
) as dag:
    with TaskGroup(group_id='canary_transformations') as canary_transformations_group:
        dbt_task = DbtTask(
            task_id='run_canary_transformations',
            profile='your_db_profile',
            project_dir='/path/to/dbt/project',
            select=['+model_name'],
            schema='sampled_schema',
            do_run_tests=True,
            retries=3,
            retry_delay=timedelta(minutes=5),
        )

    validation_task = PythonOperator(
        task_id='validate_output',
        python_callable=validate_output,
        op_kwargs={
            'expected_results_path': '/path/to/expected/results.csv',
            'output_path': '/path/to/output/directory',
        },
    )

    monitoring_task = BashOperator(
        task_id='monitor_transformations',
        bash_command="echo 'Monitoring transformations...'",
        trigger_rule=TriggerRule.ALL_SUCCESS,
    )

    dbt_task >> validation_task >> monitoring_task
```

**Data:**

```json
{
  "expected_results_path": "/path/to/expected/results.csv",
  "output_path": "/path/to/output/directory"
}
```

