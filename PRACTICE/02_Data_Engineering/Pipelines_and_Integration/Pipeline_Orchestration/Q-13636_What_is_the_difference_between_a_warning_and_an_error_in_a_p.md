---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a warning and an error in a pipeline and should a warning stop the run?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Handling Non-critical Task Failures](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Orchestration/Handling_Non-critical_Task_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data pipelines, both warnings and errors are critical signals that can impact the health of your workflow. However, they differ significantly in their severity and how they affect the overall execution of the pipeline. Warnings typically indicate issues that might not immediately halt the process but could lead to problems if left unchecked. Errors, on the other hand, represent genuine failures that prevent the pipeline from completing its intended tasks.

To address your question specifically: a warning should generally not stop the run of a pipeline, whereas an error usually does. Let's break down why:

Severity and Impact: Warnings are often used to flag potential issues or areas for improvement without halting operations. For example, if a Spark task is taking longer than expected, emitting a warning can help monitor performance but doesn't necessarily stop the process. Conversely, an error is indicative of a serious issue that prevents further execution, such as a missing file or an unexpected data format.

#### Example Scenario
s: Imagine you have a pipeline that processes log files and writes summaries to a database. If some logs are in an unrecognized format but still partially parseable, the system might emit a warning indicating that not all records were processed correctly. This allows the pipeline to continue running so other important tasks can be completed without being disrupted.

Graceful Degradation: By allowing warnings to pass through, pipelines maintain their ability to process as much data as possible under suboptimal conditions. For instance, in a real-time streaming scenario, it might be more beneficial to log and handle the warning while still processing ongoing data rather than stopping all operations due to one issue.

Failure Modes: If warnings were treated as errors, the pipeline could become overly sensitive to minor issues, leading to premature halts or unnecessary restarts. This would introduce additional complexity in managing alerts and could slow down overall workflow efficiency.

Best Practices: In practice, a good data pipeline should be designed with both warning and error handling mechanisms. Warnings can help in continuous improvement and monitoring of the system's health. Errors need to be carefully managed to ensure that critical processes are not interrupted unnecessarily.

In summary, while warnings signal potential issues for review, they do not typically halt the pipeline's execution. Errors, however, represent genuine failures that must stop the process to avoid corrupting data or propagating incorrect results downstream.

**Comparison:**

```
Warning	Error
Continues pipeline operation	Stops pipeline operation
Indicates potential issues for review	Represents genuine failures that must be addressed
May log and continue processing partially incorrect data	Requires immediate action to prevent corruption or incorrect results
```

**Code:**

```python
# Example of handling warnings and errors in a data processing function using Airflow

def process_logs(log_path):
    try:
        # Simulate reading logs
        with open(log_path, 'r') as file:
            logs = file.readlines()

        # Process each log line
        for log in logs:
            if "unrecognized" in log:
                logger.warning("Log contains unrecognized format: {}".format(log))
            else:
                process_log(log)

    except FileNotFoundError as e:
        logger.error("File not found: {}", e)
        raise  # Raising the error to stop the pipeline
```

**Example input / output:**

```text
Input:
- Process logs located at "/path/to/logs"

Output (Dry-run):
- Warnings logged for unrecognized log formats.
- Errors logged and re-raised to stop the pipeline when a file is missing.

Example Pipeline Execution:
1. Logs are read from "/path/to/logs".
2. A warning is logged for one line containing an unrecognized format.
3. The remaining logs are processed normally.
4. An error is raised due to the non-existent file, stopping further execution.
```

**Data:**

```json
{
  "log_path": "/path/to/logs",
  "expected_logs": [
    "2023-10-01T08:00:00 INFO - Starting processing",
    "2023-10-01T09:00:00 WARNING - Log contains unrecognized format: {'timestamp': '2023-10-01T09:30:00', 'message': 'Unexpected data'}",
    "2023-10-01T10:00:00 ERROR - File not found: /path/to/missing/log"
  ]
}
```

**Flow:**

1. Recognized format
2. Unrecognized format
3. Missing file
4. Start
5. Check Log Format
6. Process Log
7. Log Warning
8. Handle Error
9. Stop Pipeline

