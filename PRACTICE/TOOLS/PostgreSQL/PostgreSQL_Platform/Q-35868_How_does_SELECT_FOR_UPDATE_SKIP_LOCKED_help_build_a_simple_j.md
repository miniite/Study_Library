---
type: "practice"
difficulty: "Expert"
---

# How does SELECT ... FOR UPDATE SKIP LOCKED help build a simple job queue in PostgreSQL?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deadlocks and Lock Ordering](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Deadlocks_and_Lock_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To use SELECT ... FOR UPDATE SKIP LOCKED to build a simple job queue in PostgreSQL, we need to understand how this feature can help us efficiently manage concurrent access to jobs while ensuring fairness and avoiding deadlocks. This method is particularly useful in scenarios where multiple workers might concurrently try to pick up the same job.

#### Triage

The question asks about using SELECT ... FOR UPDATE SKIP LOCKED for building a simple job queue. The core of this feature lies in its ability to lock rows while still allowing other transactions to continue operating on unselected rows, ensuring that only one worker can process each job at a time. We will need to consider the following:

Concurrency Control: How does FOR UPDATE ensure that multiple workers don't pick up the same job?
Job Processing: What happens when a worker processes the selected job?
Fairness and Deadlocks: How can we manage fairness without causing deadlocks?

#### First-Principles Solution

Table Schema:

SQL
CREATE TABLE jobs (
    id SERIAL PRIMARY KEY,
    status VARCHAR(50) CHECK (status IN ('QUEUED', 'IN_PROGRESS', 'COMPLETED')),
    payload JSONB,
    processed_by TEXT
);

Worker Logic:

A worker will SELECT a job in the QUEUED state, lock it with FOR UPDATE, and then process it.
If no jobs are available or there is a timeout due to lack of work, the transaction can be rolled back without affecting other workers.

Processing Logic:

SQL
BEGIN;
-- Try to select a job for update while ensuring we skip locked rows
SELECT id FROM jobs WHERE status = 'QUEUED' FOR UPDATE SKIP LOCKED LIMIT 1;

IF NOT FOUND THEN
    -- No jobs available, release the lock and exit
    ROLLBACK;
    RETURN; -- Exit this worker instance
END IF;

-- Process the job
UPDATE jobs SET status = 'IN_PROGRESS', processed_by = current_user WHERE id = <selected_id>;

-- Perform job processing logic here

-- Mark the job as completed
UPDATE jobs SET status = 'COMPLETED' WHERE id = <selected_id> AND processed_by = current_user;

COMMIT;

Concurrency Control:

FOR UPDATE ensures that once a worker locks a row, no other workers can lock or update that same row until the first worker completes its transaction.
SKIP LOCKED allows other transactions to continue processing jobs while ensuring fairness among workers.

#### Closing Summary

Using SELECT ... FOR UPDATE SKIP LOCKED in PostgreSQL is an elegant way to build a simple job queue. It ensures that each job can only be processed by one worker at a time, without blocking other jobs from being picked up by another worker. The combination of FOR UPDATE and SKIP LOCKED provides a fair and efficient mechanism for handling concurrency.

#### Common Mistakes/Follow-Ups
Incorrect Transaction Handling: Forgetting to properly handle the transaction in case no job is available.
Deadlock Risk: Not managing transactions carefully can lead to deadlocks if not all paths are considered (e.g., what happens if a job gets marked as IN_PROGRESS but an error occurs before committing).
Lock Contention: High contention for locks might impact performance; monitoring and tuning lock usage is essential.

