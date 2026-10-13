---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect that a scheduled job ran but processed zero rows due to a wrong date filter?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distribution Change Alerts](<../../../../02_Data_Engineering/Data_Quality/Data_Observability/Distribution_Change_Alerts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of detecting whether a scheduled job has run but processed zero rows due to a misconfigured date filter, we need to implement a robust data observability strategy that includes logging and alerting mechanisms. The core idea is to ensure that we can quickly identify such issues by analyzing logs and monitoring metrics.

The question essentially probes our ability to design and implement a comprehensive logging system and monitoring setup. Key assumptions include the availability of job logs, the ability to set up alerts based on specific conditions, and the presence of a robust data governance framework.

First, let's break down the problem from first principles:

Identify the Job Execution Logs: We need to ensure that every scheduled job execution writes its logs to a central logging system.
Capture the Date Filter Condition: The logs must capture whether the date filter was applied correctly and if it excluded all rows.
Set Up Metrics for Row Count: Monitor the row count processed by each job execution.
Implement Alerting Mechanisms: Configure alerts that trigger when a job runs but processes zero rows.

Here's a step-by-step solution:

Capture Logs:

Ensure that your scheduled jobs log their start and end times, date filters used, and the number of rows processed or failed.

#### Example
Log Entry: Job Execution ID: 2023-09-01_1200; Date Filter Used: '2023-08-30'; Rows Processed: 0

Metric Collection:

Set up a metrics collection system to track the number of rows processed by each job execution.
Use an aggregation layer (e.g., Prometheus) to gather and store these metrics.

Alerting Configuration:

Define alert rules in your monitoring tool (e.g., Grafana) that trigger when a job is executed but processes zero rows.

#### Example
Alert Rule: Job Executions with 0 Rows Processed for More Than 1 Hour

Continuous Monitoring and Logging:

Use tools like ELK Stack or Splunk to centralize logs and enable real-time monitoring.
Ensure that all environments (dev, staging, prod) are consistently set up with these logging and alerting mechanisms.

Data Governance Framework:

Integrate data governance practices to ensure that the date filters and other parameters are validated before job execution.
Use tools like Apache Airflow or Kubernetes for job scheduling, which can be configured to fail fast if any preconditions (like incorrect date filters) aren't met.

Now, let's summarize the solution:

Implement a logging system to capture detailed information about each job execution.
Set up metrics to track row counts and other relevant performance metrics.
Configure alerting mechanisms to notify stakeholders when zero rows are processed.
Use data governance practices to ensure that jobs are properly validated before running.

#### Common mistakes
include not having a centralized logging system, ignoring the importance of metrics collection, or failing to set up effective alerting. Always validate your assumptions and ensure all steps are thoroughly tested in both development and production environments.

**Comparison:**

| Job ID        | Date Filter Used     | Rows Processed |
|---------------|---------------------|----------------|
| 2023-09-01_1200 | '2023-08-30'         | 0              |

**Code:**

```python
# Example Python Code for Logging Job Execution Details

import logging
from datetime import datetime

def log_job_execution(job_id, date_filter_used, rows_processed):
    # Configure logging
    logger = logging.getLogger('job_logger')
    logger.setLevel(logging.INFO)

    # Create a file handler and set the log level
    handler = logging.FileHandler('job_logs.log')
    handler.setLevel(logging.INFO)

    # Create a formatter and set the format
    formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
    handler.setFormatter(formatter)

    # Add the handlers to the logger
    logger.addHandler(handler)

    # Log job execution details
    if rows_processed == 0:
        message = f"Job Execution ID: {job_id}; Date Filter Used: '{date_filter_used}'; Rows Processed: {rows_processed}"
        logger.info(message)
    else:
        message = f"Job Execution ID: {job_id}; Date Filter Used: '{date_filter_used}'; Rows Processed: {rows_processed} rows"
        logger.info(message)

# Example Usage
log_job_execution('2023-09-01_1200', '2023-08-30', 0)
```

**Example input / output:**

```text
Input:
Execution Details: Job ID - 2023-09-01_1200, Date Filter Used - '2023-08-30', Rows Processed - 0

Output:
Job Execution ID: 2023-09-01_1200; Date Filter Used: '2023-08-30'; Rows Processed: 0
```

**Data:**

```json
{
  "job_id": "2023-09-01_1200",
  "date_filter_used": "2023-08-30",
  "rows_processed": 0,
  "log_message": "Job Execution ID: 2023-09-01_1200; Date Filter Used: '2023-08-30'; Rows Processed: 0"
}
```

**Flow:**

1. Yes
2. No
3. >0
4. 0
5. Job Execution Starts
6. Run Job
7. Date Filter Valid?
8. Process Rows
9. Log Error; Alert Team
10. Check Row Count
11. Success
12. Log Zero Rows; Alert Team

