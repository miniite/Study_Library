---
type: "practice"
difficulty: "Intermediate"
---

# You maintain a Python glue job in an Airflow DAG that intermittently marks itself successful while the underlying task failed. How would you fix the exit-status handling and make the operator report failure correctly?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Airflow Tasks](<../../../../TOOLS/Python/Python_Language/Idempotent_Airflow_Tasks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where a Python operator in your Airflow DAG reports success despite failing, we need to ensure that the on_failure_callback is properly configured and that our task's exit status is handled appropriately. This involves reviewing the task's code for any premature exits or return values, setting up proper logging, and potentially customizing how Airflow handles the operator's failure.

#### Triaging the Question

The core of the problem lies in ensuring that the Python operator accurately reports its failure status to Airflow. The question is probing your understanding of how on_failure_callback works, how exit statuses are managed in tasks, and the importance of proper error handling within the task itself.

#### First-Principles Solution
Review Task Code: Ensure that the code inside the Python operator either returns an appropriate status or raises an exception.
Implement on_failure_callback: Use this callback to explicitly set a failure state if any errors are encountered.
Logging and Debugging: Enhance logging within the task to capture and report detailed error information.
Detailed Steps

Review Task Code

Examine the execute method of your Python operator. Ensure it handles all possible exceptions and returns an appropriate exit status or raises a custom exception.
PYTHON
from airflow.exceptions import AirflowException

def execute(self, context):
    try:
        # Your task logic here
        result = self.run_task()
        if result == "failure":
            raise AirflowException("Task failed with custom message")
    except Exception as e:
        logger.error(f"Task encountered an error: {str(e)}", exc_info=True)
        raise AirflowException(str(e))

    return "success"

Implement on_failure_callback

Define a callback that will be invoked if the task fails.
PYTHON
def failure_callback(context, exception):
    logging.error(f"Task {context['task_instance'].task_id} failed: {exception}")
    # Optionally, you can set custom XCom or log extra details

with dag:
    task = PythonOperator(
        task_id="example_task",
        python_callable=execute,
        on_failure_callback=failure_callback
    )

Logging and Debugging

Enhance logging to capture detailed information about the failure.
PYTHON
import logging

logger = logging.getLogger(__name__)

def execute(self, context):
    try:
        # Your task logic here
        result = self.run_task()
        if result == "failure":
            raise AirflowException("Task failed with custom message")
    except Exception as e:
        logger.error(f"Task encountered an error: {str(e)}", exc_info=True)
        raise AirflowException(str(e))

    return "success"

#### Closing Summary

By ensuring the Python operator properly handles exceptions, sets appropriate exit statuses, and uses on_failure_callback, we can make sure that the task reports failure correctly. Enhancing logging helps in diagnosing issues during runtime.

#### Common Mistakes/Follow-Ups
Mistake: Not handling all exceptions within the task.

#### Solution
Use try-except blocks to catch and handle all possible exceptions gracefully.
Mistake: Failing to set the correct exit status or raising an exception.

#### Solution
Ensure that any failure is explicitly reported using raise AirflowException.

#### Follow-Up
How would you test this setup?
You can mock the task logic and verify that it correctly reports success/failure based on predefined conditions.

**Code:**

```

from airflow.exceptions import AirflowException

def execute(self, context): try: # Your task logic here result = self.run_task() if result == "failure": raise AirflowException("Task failed with custom message") except Exception as e: logger.error(f"Task encountered an error: {str(e)}", exc_info=True) raise AirflowException(str(e))

CODE
Copy
return "success"
CODE
Copy

```python
import logging

logger = logging.getLogger(__name__)

def failure_callback(context, exception):
    logging.error(f"Task {context['task_instance'].task_id} failed: {exception}")
```

