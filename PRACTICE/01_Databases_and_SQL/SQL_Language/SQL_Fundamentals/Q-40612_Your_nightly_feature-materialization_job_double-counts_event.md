---
type: "practice"
difficulty: "Deep"
---

# Your nightly feature-materialization job double-counts events on the day the pipeline restarted mid-run. How would you make the SQL materialization idempotent and safely re-runnable?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Rollups](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of double-counting events due to a restart in your nightly feature-materialization job, we need to ensure that our SQL queries are idempotent. This means that running them multiple times should have no additional effect beyond their intended operation. We'll achieve this by leveraging database transactions and unique event identifiers.

Firstly, let's define the problem: the job processes events from a source table and updates feature tables, but it may restart mid-run due to various reasons (e.g., system crashes, scheduled maintenance). When it resumes, it reprocesses some of the already processed events, leading to double-counting.

#### Triaging the Question

The interviewer is probing:

Idempotency in SQL: How you can ensure that a query or set of queries produces the same result every time they are executed.
Transaction Management: Understanding how to use transactions to handle partial completion and restarts gracefully.
Error Handling and Recovery: Ensuring the job handles failures and retries without introducing duplicates.

#### Assumptions
We have a source table with events that need materialization into feature tables.
The job can be interrupted during execution, leading to potential double-counting on resume.
Each event in the source table has a unique identifier (e.g., event_id).
Step-by-Step Solution

Identify the Issue: Double-counting occurs because the job processes events multiple times due to restarts.

Use Transactional Processing:

Wrap each processing step in a transaction that ensures all operations are atomic and consistent.
Use unique identifiers (e.g., event_id) to track which events have already been processed.

Implement Idempotent Logic:

Before processing an event, check if it has already been processed by querying the feature table using the event identifier.
If the event is not found or the features are different, proceed with materialization; otherwise, skip it.

Handle Partial Completion:

Use checkpoints to save the state of the job before it gets interrupted.
When resuming, only process events that have not been processed since the last checkpoint.
Detailed Solution
Step 1: Checkpoint Management

We'll use a separate table or column in the feature tables to track which events have already been processed. This can be done using a unique index on event_id.

SQL
-- Example of tracking processed events in a table
CREATE TABLE if not exists processed_events (
    event_id BIGINT PRIMARY KEY,
    processed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
Step 2: Materialization Logic

When processing an event, first check if it has already been processed:

SQL
INSERT INTO feature_table (event_id, feature_1, feature_2)
SELECT e.event_id, e.feature_1, e.feature_2
FROM events e
LEFT JOIN processed_events pe ON pe.event_id = e.event_id
WHERE pe.event_id IS NULL;

This query will only insert new records into feature_table if the event has not been processed before. The join with processed_events ensures that we're only inserting or updating features for unprocessed events.

Step 3: Handling Partial Completion and Restart

To handle partial completion, we can use a checkpoint mechanism:

SQL
-- Insert checkpoint after each batch of events is processed
INSERT INTO checkpoints (checkpoint_time) VALUES (NOW());

When resuming the job, you would only process events since the last checkpoint timestamp.

#### Closing Summary

By implementing these steps, we ensure that our SQL materialization logic is idempotent and can safely be re-run without causing double-counting. This approach leverages transactional processing, unique identifiers for event tracking, and a robust checkpoint mechanism to handle partial completion gracefully.

#### Common Mistakes/Follow-Ups
Forgetting to Track Processed Events: Not maintaining an processed_events table or column can lead to double-processing.
Incorrect Checkpoint Handling: If checkpoints are not handled correctly (e.g., missing timestamps, incorrect logic), it may lead to processing events that were already processed since the last checkpoint.

Do you have any follow-up questions on this approach?

**Code:**

```sql
-- Example of tracking processed events in a table
CREATE TABLE if not exists processed_events (
    event_id BIGINT PRIMARY KEY,
    processed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Materialization logic
INSERT INTO feature_table (event_id, feature_1, feature_2)
SELECT e.event_id, e.feature_1, e.feature_2
FROM events e
LEFT JOIN processed_events pe ON pe.event_id = e.event_id
WHERE pe.event_id IS NULL;

-- Inserting a checkpoint after each batch of events is processed
INSERT INTO checkpoints (checkpoint_time) VALUES (NOW());
```

**Example input / output:**

```text
-- Example input for events to be processed

INSERT INTO events (event_id, feature_1, feature_2) VALUES
(1, 10, 20),
(2, 30, 40),
(3, 50, 60);

-- Checkpoint after processing
INSERT INTO checkpoints (checkpoint_time) VALUES ('2023-10-01T12:00:00Z');

-- Processing events with idempotent logic

INSERT INTO feature_table (event_id, feature_1, feature_2)
SELECT e.event_id, e.feature_1, e.feature_2
FROM events e
LEFT JOIN processed_events pe ON pe.event_id = e.event_id
WHERE pe.event_id IS NULL;

-- Check if any new events were inserted

SELECT * FROM feature_table WHERE event_id IN (1, 2, 3);
```

**Data:**

```json
{
    "events": [
        {"event_id": 1, "feature_1": 10, "feature_2": 20},
        {"event_id": 2, "feature_1": 30, "feature_2": 40},
        {"event_id": 3, "feature_1": 50, "feature_2": 60}
    ],
    "processed_events": [
        {"event_id": 1},
        {"event_id": 3}
    ],
    "checkpoint": {
        "last_checkpoint_time": "2023-10-01T12:00:00Z"
    }
}
```

