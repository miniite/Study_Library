---
type: "practice"
difficulty: "Deep"
---

# How would you implement a dead-letter and reprocessing path for LLM jobs that a FastAPI-triggered worker repeatedly fails to complete, so they do not retry forever?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Long-running Jobs and Async Patterns](<../../../../TOOLS/Python/FastAPI/Long-running_Jobs_and_Async_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge of implementing a dead-letter and reprocessing path for LLM jobs in a FastAPI-based system where worker tasks fail repeatedly, we need to establish a mechanism that logs failing jobs into a "dead-letter queue" (DLQ), then periodically retrieves these jobs to retry them. This approach ensures that the system doesn't get stuck endlessly retrying failed jobs but also prevents losing potentially important or urgent work.

This solution will be implemented using FastAPI for triggering and managing tasks, along with a background task worker (like Celery) for handling job execution. The key components are:

A database to store job metadata.
An ElasticSearch instance as the DLQ.
Periodic checks by a scheduler to reprocess jobs from the DLQ.

Let's dive into the implementation details.

#### Triage
and Assumptions
Database: We assume PostgreSQL for storing job metadata due to its robust features like ACID transactions, which are crucial for managing retries.
Worker Framework: Celery is used as it offers a reliable task queue with built-in support for retries and result storage.
Scheduler: A cron job or an event-driven scheduler (e.g., Apache Airflow) will be responsible for periodically checking the DLQ.
Step-by-Step Solution

Job Scheduling:

Use FastAPI to define API endpoints that trigger tasks.
These APIs enqueue jobs into a Celery task queue with retries enabled.

Worker Implementation:

Define Celery tasks that perform the LLM job execution logic.
Implement retry strategies within these tasks, including exponential backoff and jitter to avoid overwhelming the system during failures.

Dead-Letter Queue (DLQ):

Configure Celery to send jobs to a DLQ after all retries fail.
Use an ElasticSearch instance as the DLQ because it can store logs and metadata efficiently and provides powerful querying capabilities for retrieval.

Periodic Job Rescheduling:

Implement a scheduler that periodically checks the DLQ to retrieve and reprocess jobs.
This can be done using a cron job or by leveraging an event-driven architecture where the scheduler triggers based on job completion status updates.
Full Solution
Database Schema
SQL
CREATE TABLE jobs (
    id SERIAL PRIMARY KEY,
    task_id VARCHAR(255) NOT NULL UNIQUE,  -- Unique identifier for each Celery task
    llm_job_data JSONB,  -- Data specific to the LLM job
    status VARCHAR(50),  -- Status of the job (e.g., "failed", "pending")
    retry_count INTEGER DEFAULT 0,        -- Number of retries attempted
    last_retried_at TIMESTAMP            -- Timestamp of the last retry attempt
);
Celery Task Definition
PYTHON
from celery import shared_task

@shared_task(bind=True)
def llm_job(self, job_data):
    # Perform LLM job execution logic here
    try:
        result = execute_llm(job_data)  # Placeholder for actual LLM execution
        if result.status_code != 200:  # Example condition for failure
            raise Exception("LLM API failed")
    except Exception as e:
        self.retry(countdown=60 * (2 ** self.request.retries), exc=e)
Periodic Rescheduling Logic
PYTHON
from celery import Celery
import schedule, time

app = Celery('reschedule', broker='redis://localhost:6379/0')

@app.task(bind=True)
def reschedule_jobs(self):
    with app.app_session() as session:
        # Query jobs that failed and need reprocessing
        dlq_jobs = session.query(Job).filter_by(status='failed').all()
        for job in dlq_jobs:
            execute_llm_job.delay(job.llm_job_data)

# Schedule the rescheduling task to run every hour
schedule.every(1).hours.do(reschedule_jobs)
while True:
    schedule.run_pending()
    time.sleep(60)
Sample Data
JSON
{
  "task_id": "task_123",
  "llm_job_data": {
    "prompt": "What is the capital of France?",
    "model_version": "v2.0"
  },
  "status": "failed",
  "retry_count": 3,
  "last_retried_at": "2023-10-17T14:48:00Z"
}

#### Closing Summary
and Common Mistakes

This approach ensures that jobs are retried intelligently, balancing between preventing infinite retries and not losing important work. Key takeaways include:

Using a DLQ to segregate failed jobs from active tasks.
Implementing exponential backoff with jitter in retry strategies.
Periodically rescheduling jobs using a scheduler to handle dead-lettered tasks.

#### Common mistakes
to avoid:

Infinite Retries: Ensure there is a mechanism to mark and skip certain jobs after a maximum number of retries.
Resource Exhaustion: Handle large volumes of failed jobs efficiently by configuring the DLQ appropriately.
Data Consistency: Ensure job states are correctly updated in the database to avoid conflicts or infinite loops.

**Comparison:**

```
Task ID	LLM Job Data	Status	Retry Count	Last Retried At
task_123	{"prompt": "What is the capital of France?", "model_version": "v2.0"}	failed	3	2023-10-17T14:48:00Z
```

**Code:**

```

from celery import shared_task

@shared_task(bind=True) def llm_job(self, job_data): # Perform LLM job execution logic here try: result = execute_llm(job_data) # Placeholder for actual LLM execution if result.status_code != 200: # Example condition for failure raise Exception("LLM API failed") except Exception as e: self.retry(countdown=60 * (2 ** self.request.retries), exc=e)

CODE
Copy

```python
from celery import Celery

app = Celery('reschedule', broker='redis://localhost:6379/0')

@app.task(bind=True)
def reschedule_jobs(self):
    with app.app_session() as session:
        # Query jobs that failed and need reprocessing
        dlq_jobs = session.query(Job).filter_by(status='failed').all()
        for job in dlq_jobs:
            execute_llm_job.delay(job.llm_job_data)

schedule.every(1).hours.do(reschedule_jobs)
while True:
    schedule.run_pending()
    time.sleep(60)
```

**Example input / output:**

```text
{
  "task_id": "task_123",
  "llm_job_data": {
    "prompt": "What is the capital of France?",
    "model_version": "v2.0"
  },
  "status": "failed",
  "retry_count": 3,
  "last_retried_at": "2023-10-17T14:48:00Z"
}
```

**Data:**

```sql
CREATE TABLE jobs (
    id SERIAL PRIMARY KEY,
    task_id VARCHAR(255) NOT NULL UNIQUE,  -- Unique identifier for each Celery task
    llm_job_data JSONB,  -- Data specific to the LLM job
    status VARCHAR(50),  -- Status of the job (e.g., "failed", "pending")
    retry_count INTEGER DEFAULT 0,        -- Number of retries attempted
    last_retried_at TIMESTAMP            -- Timestamp of the last retry attempt
);
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Trigger LLM Job
6. Queue Job in Celery TaskQueue
7. Job Succeeds?
8. Mark Job as Complete
9. Attempt Retry
10. Retry Count Exceeded?
11. Log to Dead-Letter Queue(DLQ)
12. Reschedule Jobs from DLQ

