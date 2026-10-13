---
type: "practice"
difficulty: "Basic"
---

# What information would you look at in the Airflow logs when a task fails and you don't know why?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Non-critical Task Failures](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a task fails unexpectedly in an Airflow pipeline, the first step is to understand what happened from the logs. As a Data Engineer fresher or mid-level engineer, I would look at several key pieces of information from the Airflow UI and logs to diagnose the issue.

The primary areas to check include:

The task instance details
The log output for the failed task
The task's upstream dependencies and their status

From first principles, we need to systematically approach this by examining each component to identify where things may have gone wrong.

**Comparison:**

```
Task Instance Details	Log Output
Task ID	123456
Dag ID	dag_name
Execution Date	2023-09-15T14:30:00Z
```

**Code:**

```python
# Example of how to access Airflow logs programmatically (Python API)
from airflow.models import TaskInstance
ti = TaskInstance(task=task, execution_date=datetime.utcnow())
print(ti.log)
```

