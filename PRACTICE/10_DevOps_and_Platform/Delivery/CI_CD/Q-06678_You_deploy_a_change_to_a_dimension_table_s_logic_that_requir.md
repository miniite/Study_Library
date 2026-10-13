---
type: "practice"
difficulty: "Deep"
---

# You deploy a change to a dimension table's logic that requires re-processing history; how would you sequence the backfill and the code deploy so downstream marts never read a half-rebuilt table?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Expand and Contract Migrations](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Expand_and_Contract_Migrations.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deploying changes to a dimension table in a data warehouse, ensuring consistency across historical and current data is crucial. The challenge here is sequencing the backfill (re-processing of historical data) and code deployment so that downstream marts never see an inconsistent state—i.e., a partially rebuilt dimension table.

#### Triaging the Question

The key aspects this question probes are:

Understanding Data Consistency: Ensuring historical data integrity.
Deployment Strategy: Coordinating multiple steps (backfill and code deploy) to avoid downtime or inconsistency.
CI/CD Practices: Leveraging Continuous Integration and Continuous Deployment principles in a complex environment.

#### Assumptions
The dimension table is part of an existing ETL process.
Downstream marts rely on the complete and consistent state of the dimension table.
The change involves logic that affects historical records, necessitating backfill.

#### First Principles Solution

Preparation Phase:

Notify downstream teams about upcoming changes to ensure they are aware and can plan accordingly.
Schedule the deployment time window during off-peak hours to minimize impact on end users.

Backfill Execution:

Initiate backfill jobs that process historical data according to the new logic.
Ensure these jobs run in parallel but independently, avoiding any overlap with current ETL processes.
Monitor each backfill job for errors and retry mechanisms if necessary.

Code Deployment:

Deploy the updated codebase responsible for dimension table logic changes to all environments (staging, QA, production).
Ensure that during deployment, a rollback strategy is in place just in case something goes wrong.
Use blue-green deployments or canary releases where possible to minimize disruption.

Post-Deployment Validation:

Validate the new code against historical data using predefined checks and tests.
Run integration tests with downstream marts to ensure they receive consistent and complete datasets.

Transition Phase:

Once validated, switch off old dimension table versions in the ETL processes.
Gradually decommission the old logic and backfill jobs.
Communicate the successful transition to stakeholders.

#### Closing Summary

By carefully sequencing the backfill and code deploy phases, we can ensure that downstream systems always see a consistent state of data. The key is to prepare thoroughly, execute in stages, validate rigorously, and communicate effectively throughout the process.

#### Common Mistakes/Follow-Ups
Notifying Stakeholders: Failure to inform downstream teams about changes might lead to confusion.
Improper Backfill Execution: Overlapping backfill jobs or errors during processing can result in data inconsistencies.
Insufficient Validation: Lack of thorough validation against historical data can introduce bugs that only surface post-deployment.
Triangulation and Confirmation

This approach ensures a deep understanding of CI/CD principles applied to complex, data-driven environments. Careful planning and execution are crucial for maintaining system integrity during changes.

**Comparison:**

| Date       | Job ID | Status     |
|------------|--------|------------|
| 2023-01-01 | 1      | Completed  |
| 2023-01-02 | 2      | InProgress |

**Code:**

```python
# Example Python script for backfill job initiation
def initiate_backfills():
    # Logic to start backfill processes in parallel
    backfill_jobs = [
        {"date": "2023-01-01", "job_id": 1},
        {"date": "2023-01-02", "job_id": 2}
    ]

    for job in backfill_jobs:
        start_backfill(job["date"], job["job_id"])

# Function to start a backfill process
def start_backfill(date, job_id):
    print(f"Starting backfill for {date} with job ID: {job_id}")
```

**Example input / output:**

```text
Input:
initiate_backfills()

Output:
Starting backfill for 2023-01-01 with job ID: 1
Starting backfill for 2023-01-02 with job ID: 2

Validation Output:
{
  "backfills": [
    {"date": "2023-01-01", "status": "Completed"},
    {"date": "2023-01-02", "status": "Completed"}
  ]
}
```

**Data:**

```json
{
  "backfills": [
    {"date": "2023-01-01", "status": "Completed"},
    {"date": "2023-01-02", "status": "InProgress"}
  ]
}
```

**Flow:**

1. Yes
2. Prepare
3. Notify Stakeholders
4. Schedule Deployment
5. Initiate Backfills
6. Monitor Jobs
7. Validate Code
8. Switch Off Old Logic
9. Communicate Success

