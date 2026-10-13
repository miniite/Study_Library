---
type: "practice"
difficulty: "Intermediate"
---

# What is templating with Jinja in Airflow and how might you inject the execution date into a task?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Orchestration vs Processing](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Orchestration_vs_Processing/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, data pipelines often require dynamic values based on runtime contexts such as timestamps or environment variables. Templating is one of the key mechanisms in Apache Airflow to achieve this flexibility. Specifically, Jinja templating allows us to inject dynamic content into task parameters using Python-like syntax. In this answer, we will discuss how Jinja templating works and demonstrate injecting the execution date into a task.

Jinja templating in Airflow is powered by the Jinja2 library, which provides powerful template logic capabilities. When defining tasks, you can use Jinja templates to inject dynamic values such as the current execution date or environment variables into task parameters. For example, this allows you to dynamically generate filenames or URLs based on the execution time.

To illustrate, let's consider a scenario where we need to ingest data from a source system and store it in a database table named after the execution date. We will define an Airflow DAG that uses Jinja templating to inject the current execution date into the task's parameter.

Here’s how you would do it:

Define the template string with placeholders.
Use Airflow's macros and variables to dynamically populate these placeholders at runtime.

**Code:**

```

from airflow import DAG from datetime import timedelta from airflow.macros import ds_format, ds_add from airflow.utils.dates import days_ago

Define the DAG with a start date and schedule interval

default_args = { 'owner': 'airflow', 'depends_on_past': False, 'start_date': days_ago(1), }

dag = DAG( 'example_jinja_dag', default_args=default_args, description='A simple example of templating with Jinja in Airflow.', schedule_interval=timedelta(hours=1), )

Define a task that uses Jinja template to inject the execution date

@dag.task(multiple_outputs=True) def ingest_data(execution_date): return { 'file_name': f"data_{ds_format(execution_date, '%Y-%m-%d')}.csv", 'target_table': f'raw_data_{ds_format(execution_date, '%Y%m%d')}" }

Use the templated task in a downstream task

@dag.task() def process_data(task_output): file_name = task_output['file_name'] target_table = task_output['target_table']

CODE
Copy
# Process data and load into target table
pass  # Placeholder for actual processing logic

ingest_task = ingest_data(execution_date='{{ execution_date }}') process_task = process_data(ingest_task)

CODE
Copy

```python
# The above code uses Jinja template syntax to inject the `execution_date` into task parameters.
# For example, `ds_format(execution_date, '%Y-%m-%d')` will return a string like '2023-10-05'.
```

**Example input / output:**

```text
graph TD;
    A[Ingest Data Task] --> B{Execution Date};
    B --> C{Format Date as 'YYYY-MM-DD'};
    C --> D[Generated File Name];
    D --> E{Execute Processing Task};

    F[Process Data Task] --> G{Target Table Name};
    G --> H{Load into Database};
```

**Data:**

```json
{
    "file_name": "data_2023-10-05.csv",
    "target_table": "raw_data_20231005"
}
```

