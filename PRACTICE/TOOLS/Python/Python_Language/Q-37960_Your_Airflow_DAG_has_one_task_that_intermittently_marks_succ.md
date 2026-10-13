---
type: "practice"
difficulty: "Deep"
---

# Your Airflow DAG has one task that intermittently marks success while the underlying Spark job failed. How would you fix the exit-status handling so the operator reports failure?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Airflow Tasks](<../../../../TOOLS/Python/Python_Language/Idempotent_Airflow_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of an intermittent success marking in your Airflow DAG when the underlying Spark job fails, we need to ensure that our Python-based Airflow operators properly handle and propagate exit statuses from the Spark tasks. This involves understanding how the SparkSubmitOperator works within Airflow and adjusting its configuration or customizing the task to accurately reflect the failure.

Triaging the Problem

The question is essentially about ensuring correct status handling in Airflow's SparkSubmitOperator. The core of the issue lies in making sure that when a Spark job fails, it correctly sets the exit code, which should then be picked up by the Airflow task and propagated to the DAG as a failure.

#### Assumptions
We are using Apache Airflow with version 2.x.
The problematic task is defined using SparkSubmitOperator.
The underlying Spark job occasionally returns an incorrect status (success instead of failure).

#### First Principles Solution
Understand the Exit Status: In Spark, tasks return an exit code or status that can be captured and used to determine whether a job succeeded or failed.
Configure SparkSubmitOperator: Adjust the SparkSubmitOperator configuration to ensure it correctly reports back the status from the Spark task.
Customize Task Handling: Write custom code within the operator's task to explicitly check for non-zero exit codes and set the task state to failure if necessary.

#### Implementation
Steps

Review Operator Configuration:

PYTHON
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

# Example of configuring the operator
spark_task = SparkSubmitOperator(
    task_id='spark_job',
    application='/path/to/spark/app.jar',  # Path to your Spark application JAR
    conf={'spark.some.config': 'value'},  # Any additional configurations
    dag=dag,
    executor_config={
        "pyFiles": "/path/to/extra_files.zip",  # Additional Python files if needed
    },
)

Check for Non-Zero Exit Codes: By default, SparkSubmitOperator does not explicitly check the exit code of the Spark job. We need to add this logic manually.

PYTHON
from airflow.models import TaskInstance

def check_exit_status(ti: TaskInstance):
    if ti.xcom_pull(task_ids='spark_job', key='return_value') != 0:
        raise ValueError("Spark job failed with non-zero exit code")

# Add the custom function to your task
spark_task.on_failure_callback = check_exit_status

Custom XComs: Ensure that the Spark application logs its exit status and makes it available via XCom.

PYTHON
from pyspark import SparkContext

def main(sc: SparkContext):
    # Your Spark job logic here
    sc.addPyFile("/path/to/extra_files.zip")
    # Example of setting an XCom value
    spark_session = SparkSession.builder.getOrCreate()
    result = 42  # Example result or error code
    spark_session.udf.register("exit_status", lambda x: str(x))
    spark_session.sql(f"SELECT exit_status({result})").collect()

if __name__ == "__main__":
    main(sc)

Task Logging: Ensure that the task logs are properly configured to capture and display any errors or non-zero exit codes.

PYTHON
logging.basicConfig(
    level=logging.DEBUG,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[logging.StreamHandler()],
)

#### Closing Summary

By configuring the SparkSubmitOperator with a custom failure callback and ensuring that the Spark application correctly sets its exit code, we can ensure that Airflow accurately reflects the success or failure of the underlying task. This approach requires careful handling of XComs and logs to capture any issues during execution.

#### Common Mistakes
Forgetting to set up an on_failure_callback in the operator configuration.
Not checking for non-zero exit codes in custom code snippets.
Ignoring proper logging, which can make debugging difficult.

**Comparison:**

| Spark Config | Value |
|--------------|-------|
| application  | /path/to/spark/app.jar |
| conf         | spark.some.config=value |
| executor_config | {"pyFiles": "/path/to/extra_files.zip"} |

**Code:**

```python
from airflow.providers.apache.spark.operators.spark_submit import SparkSubmitOperator

# Example of configuring the operator
spark_task = SparkSubmitOperator(
    task_id='spark_job',
    application='/path/to/spark/app.jar',  # Path to your Spark application JAR
    conf={'spark.some.config': 'value'},  # Any additional configurations
    dag=dag,
    executor_config={
        "pyFiles": "/path/to/extra_files.zip",  # Additional Python files if needed
    },
)

# Add the custom function to your task
def check_exit_status(ti: TaskInstance):
    if ti.xcom_pull(task_ids='spark_job', key='return_value') != 0:
        raise ValueError("Spark job failed with non-zero exit code")

spark_task.on_failure_callback = check_exit_status
```

**Example input / output:**

```text
# Example input and expected output
input_data = """
application='/path/to/spark/app.jar',
conf={'spark.some.config': 'value'},
executor_config={
    "pyFiles": "/path/to/extra_files.zip"
},
on_failure_callback=check_exit_status
"""

expected_output = """
Spark job task created with on_failure_callback.
Task will check for non-zero exit codes and set failure if present.
"""
```

**Data:**

```json
{
  "application": "/path/to/spark/app.jar",
  "conf": {
    "spark.some.config": "value"
  },
  "executor_config": {
    "pyFiles": "/path/to/extra_files.zip"
  }
}
```

**Flow:**

1. Start
2. ConfigureSparkSubmitOperator
3. Define on_failure_callback()
4. Check exit status via XCom pull
5. Set task state to failure ifnon-zero
6. End

