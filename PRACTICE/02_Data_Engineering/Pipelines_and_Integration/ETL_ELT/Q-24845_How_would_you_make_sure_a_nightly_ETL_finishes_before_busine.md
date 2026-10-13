---
type: "practice"
difficulty: "Intermediate"
---

# How would you make sure a nightly ETL finishes before business hours, and what if it does not?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Safe Re-runs of Failed Jobs](<../../../../02_Data_Engineering/Pipelines_and_Integration/ETL_ELT/Safe_Re-runs_of_Failed_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure an ETL (Extract, Transform, Load) process completes by business hours, we need to establish robust monitoring, alerting, and potentially fallback mechanisms. This involves checking job status at critical points, setting up notifications, and possibly adjusting the pipeline or scheduling based on run-time performance. If a nightly ETL does not complete before business hours, it’s crucial to understand why and take corrective actions.

**Comparison:**

```
Monitoring Interval	Action
Every 10 minutes	Check ETL job status.
After 8 hours	Take corrective actions.
```

**Code:**

```python
# Example Python code snippet for monitoring an ETL job status
import time

def monitor_etl_job(etl_job_id):
    # Placeholder function to check the status of an ETL job
    while True:
        job_status = get_etl_job_status(etl_job_id)
        if job_status == 'COMPLETED':
            print("ETL job completed successfully.")
            return
        elif job_status in ['FAILED', 'KILLED']:
            raise Exception(f"ETL job failed or was killed: {job_status}")

        # Check every 10 minutes, with a maximum of 8 hours (480 minutes) before taking action
        if time.time() - start_time > 480:
            print("ETL job did not complete within the allowed timeframe. Taking corrective actions...")
            handle_job_timeout()
            return

        time.sleep(600)
```

**Example input / output:**

```text
Input:
- ETL job ID: 12345

Output:
- Job status checks every 10 minutes (600 seconds)
- If the job completes within 8 hours, it prints a success message.
- If the job fails or is killed, an exception is raised.
- If the job does not complete within 8 hours, corrective actions are taken.
```

**Data:**

```json
{
  "etl_job_id": "12345",
  "job_status_options": ["NOT_STARTED", "RUNNING", "COMPLETED", "FAILED", "KILLED"]
}
```

**Flow:**

1. No
2. No
3. Yes
4. Start
5. Is ETL Job Completed?
6. Check if Time Exceeded 8Hours?
7. Take Corrective Actions
8. End

