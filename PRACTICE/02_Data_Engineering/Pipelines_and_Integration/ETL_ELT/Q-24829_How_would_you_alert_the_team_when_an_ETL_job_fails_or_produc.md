---
type: "practice"
difficulty: "Intermediate"
---

# How would you alert the team when an ETL job fails or produces zero rows?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Safe Re-runs of Failed Jobs](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Safe_Re-runs_of_Failed_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that the team is promptly notified when an ETL (Extract, Transform, Load) job fails or produces no rows, we need to implement a robust alerting mechanism. This involves monitoring the ETL pipeline and setting up appropriate alerts based on specific conditions. Let's break down the solution from first principles.

#### Triage

The question asks us to set up an alert system for ETL jobs that fail or produce zero rows. The key elements here are understanding the failure condition, determining what constitutes a successful job, and defining the alerting strategy. We'll assume typical monitoring practices such as database connections, job execution status, and output data volumes.

#### First-Principles Solution

Define Success and Failure Conditions:

A job is considered successful if it completes without errors and produces at least one row.
If the job fails (e.g., due to an error or unavailability) or no rows are produced, an alert should be triggered.

Implement Job Monitoring:

Use a monitoring tool or script to track the ETL job's status and output.
Common tools include Airflow with sensors, custom scripts using languages like Python, or commercial solutions like Datadog, New Relic, etc.

Alerting Strategy:

Set up alerts for both failure states (job fails) and success conditions (no rows produced).
Configure notifications via email, Slack, or other preferred channels to alert the team.

Monitoring Tool Integration:

Integrate monitoring into the ETL pipeline's existing structure.
For example, if using Apache Airflow, use Sensor operators to monitor downstream tasks.

Alert Logic Implementation:

Write a script or define a rule in your monitoring tool that checks for both job failure and zero rows produced.

#### Example
Check if the job failed or the number of rows processed is 0.

Testing:

Test the alerting mechanism by manually failing jobs and ensuring alerts are received.
Simulate scenarios where no rows are produced to verify the alert works correctly.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, setting up an alert system involves defining clear failure conditions, monitoring job status and output, and configuring notifications appropriately. Common mistakes include overlooking edge cases (e.g., partial data loads), not testing thoroughly, or using suboptimal monitoring tools that don’t provide comprehensive insights.

Common follow-ups might involve discussing how to handle different types of errors, integrating with existing logging systems, and ensuring alerting doesn't become overwhelming for the team.

**Comparison:**

| Condition       | Action            |
|-----------------|-------------------|
| Job Failed      | Send Failure Alert|
| No Rows Produced| Send Zero Row Alert|

**Code:**

```python
# Example Python script for ETL job monitoring and alerting

import sys
from datetime import datetime
import smtplib
from email.mime.text import MIMEText

def send_alert(message):
    sender = "data.alerts@example.com"
    receivers = ["team@example.com"]

    msg = MIMEText(message)
    msg['Subject'] = f"ETL Job Alert - {datetime.now()}"
    msg['From'] = sender
    msg['To'] = ", ".join(receivers)

    try:
        smtpObj = smtplib.SMTP('localhost')
        smtpObj.sendmail(sender, receivers, msg.as_string())
        print("Successfully sent email")
    except Exception as e:
        print(f"Error: unable to send email. {e}")

def main():
    # Simulate checking ETL job status
    job_status = "ERROR"
    rows_processed = 0

    if job_status == "ERROR":
        send_alert("ETL Job Failed - Check logs for details")
    elif rows_processed == 0:
        send_alert("ETL Job Produced No Rows")

if __name__ == "__main__":
    main()
```

**Example input / output:**

```text
# Example input and output for the script

def check_job_status(job_status, rows_processed):
    if job_status == "ERROR":
        return "ETL Job Failed - Check logs for details"
    elif rows_processed == 0:
        return "ETL Job Produced No Rows"

input_data = {
    "job_status": "ERROR",
    "rows_processed": 0
}

output_message = check_job_status(**input_data)
print(output_message)  # Output: ETL Job Failed - Check logs for details

input_data = {
    "job_status": "SUCCESS",
    "rows_processed": 2
}

output_message = check_job_status(**input_data)
print(output_message)  # Output: None (no alert sent as job succeeded and rows processed > 0)
```

**Data:**

```json
{
  "job_status": "ERROR",
  "rows_processed": 0,
  "expected_output": [
    {"id": 1, "name": "John Doe", "age": 30},
    {"id": 2, "name": "Jane Smith", "age": 25}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Check ETL Job Status
6. Job Failed?
7. Send Failure Alert
8. No Rows Processed?
9. Send Zero Row Alert
10. Job Successful - No Alert

