---
type: "practice"
difficulty: "Basic"
---

# What are the common task states in Airflow (like queued running success failed) and what does each mean?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Non-critical Task Failures](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Apache Airflow, understanding the various task states is crucial for managing data pipelines effectively. The primary task states you'll encounter are Queued, Running, Success, Failed, Skipped, UpstreamFailed, and RemoteQueueingError. Let's dive into each one to understand their meanings.

**Code:**

```text
# Common Airflow Task States
Queued - The task is waiting in the queue.
Running - The task is currently being executed.
Success - The task completed successfully.
Failed - The task encountered an error during execution and failed.
Skipped - The task was skipped based on a condition (e.g., a decision in the DAG).
UpstreamFailed - A dependent task upstream failed, causing this task to be skipped or fail as well.
RemoteQueueingError - An error occurred while queuing the task remotely.
```

