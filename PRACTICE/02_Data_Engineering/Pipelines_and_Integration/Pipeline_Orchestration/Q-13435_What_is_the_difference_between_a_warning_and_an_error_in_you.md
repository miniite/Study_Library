---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a warning and an error in your pipeline logs and how would you treat them differently?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Non-critical Task Failures](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, particularly when working with data pipelines using frameworks like Apache Airflow or Apache Spark, logging plays a crucial role in maintaining system health and diagnosing issues. Warnings and errors are two common types of log entries that indicate different levels of severity. In this context, warnings typically signify potential problems that do not immediately halt the pipeline but may need attention to prevent further issues. Errors, on the other hand, represent critical failures that can stop the execution or significantly impact data integrity.

To treat these differently:

Warnings: These are usually logged when a task encounters a non-critical issue. For example, if a file doesn't exist at the expected location but is optional for processing. Warnings help in identifying potential bottlenecks and areas that might need improvement without disrupting ongoing operations.
Errors: These indicate critical failures such as missing mandatory files or unexpected data formats that prevent tasks from executing correctly. Errors often halt pipeline execution to avoid producing corrupt or incorrect output.

The primary difference is the severity level and the impact on the system's operation:

Warnings are informational and can be managed by adjusting configurations or improving data management practices.
Errors require immediate intervention, such as stopping the pipeline and investigating the root cause to prevent further issues.

