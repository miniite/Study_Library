---
type: "practice"
difficulty: "Deep"
---

# How do you make a PySpark pipeline retry-safe when Airflow reruns a failed task — what state must be cleaned up on retry?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Structured Streaming in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Structured_Streaming_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with PySpark pipelines within an Airflow environment, ensuring that tasks are retry-safe is crucial to avoid data corruption or processing issues during retries. A common challenge arises when Airflow reruns a failed task, as it may attempt to process the same state twice, leading to errors such as duplicate records or inconsistent state. The key idea here is to ensure that any state or intermediate results produced by PySpark are cleanly handled and reset on retry.

To make a PySpark pipeline retry-safe in this context:

Understand Airflow’s Task Retries: Recognize how Airflow handles retries, including its default settings and hooks.
Isolate State Management: Ensure that any state or intermediate data produced by the PySpark job is managed independently from the task itself.
Implement Cleanup Mechanisms: Explicitly clean up any temporary files or states generated during the execution of a PySpark job.

Let's break down this process step-by-step:

#### Triaging the Question
What Airflow Is Probing: The core aspect here involves ensuring that retrying an Airflow task does not cause data processing issues due to state mismanagement.

#### Assumptions
Airflow is configured with retries enabled for a specific task.
PySpark code runs within the context of this retryable task.
Key Concepts: State management, job cleanup, and handling of temporary files.
Step-by-Step Solution

Identify Temporary Files and Intermediate Results:

PySpark jobs often write intermediate results or data to disk for performance reasons.
These files can become a source of issues if not properly cleaned up on retries.

Use Local State Handling with Context Managers:

Utilize Python’s tempfile module to create temporary directories and files that are automatically cleaned up when the context is exited.

#### Example
PYTHON
import tempfile

with tempfile.TemporaryDirectory() as temp_dir:
    # Perform PySpark operations here, writing intermediate results to 'temp_dir'

Manage Intermediate Data Explicitly:

Use PySpark’s write methods with appropriate parameters to control data persistence and cleanup.

#### Example
PYTHON
df.write.parquet(temp_dir, mode='overwrite')
Ensure that the mode='overwrite' parameter is used to avoid duplicate writes.

Implement Cleanup Hooks in Airflow:

Define custom Python operators or sensors that perform cleanup operations before and after tasks.

#### Example
PYTHON
from airflow.sensors.python import PythonSensor

class CleanUpOperator(BaseOperator):
    def execute(self, context):
        temp_dir = '/path/to/temp/directory'
        shutil.rmtree(temp_dir)
        return 'Cleanup complete'

# Define a sensor to clean up temporary files before retrying the task
cleanup_sensor = PythonSensor(
    task_id='cleanup_temp_files',
    python_callable=CleanUpOperator,
    retries=0,  # No need for retries here
    soft_fail=False,  # Fail immediately if cleanup fails
    timeout=300,  # 5 minutes to clean up
)

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By managing temporary files and state explicitly, you can ensure that PySpark pipelines are retry-safe in an Airflow environment. Clean up any intermediate data and use context managers to handle temporary directories.

#### Common Mistakes
Failing to clean up temporary files, leading to data corruption or processing issues on retries.
Overwriting existing data without proper mode settings.
Not defining cleanup hooks that are triggered before task retries.

#### Follow-up
questions might include discussing the trade-offs between state management and performance, or how to handle more complex state dependencies in PySpark pipelines.

**Code:**

```python
import tempfile
import shutil

with tempfile.TemporaryDirectory() as temp_dir:
    # Perform PySpark operations here, writing intermediate results to 'temp_dir'
    df.write.parquet(temp_dir, mode='overwrite')

# Define a cleanup operator
class CleanUpOperator(BaseOperator):
    def execute(self, context):
        temp_dir = '/path/to/temp/directory'
        shutil.rmtree(temp_dir)
        return 'Cleanup complete'

# Define a sensor to clean up temporary files before retrying the task
cleanup_sensor = PythonSensor(
    task_id='cleanup_temp_files',
    python_callable=CleanUpOperator,
    retries=0,  # No need for retries here
    soft_fail=False,  # Fail immediately if cleanup fails
    timeout=300,  # 5 minutes to clean up
)
```

