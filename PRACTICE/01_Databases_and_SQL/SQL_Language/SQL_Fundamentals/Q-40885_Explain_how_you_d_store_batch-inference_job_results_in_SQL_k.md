---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd store batch-inference job results in SQL keyed by (batch_id, input_row_id) so a partially-failed batch can be resumed without re-running completed rows.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Batch Updates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To store batch-inference job results in SQL while ensuring that partially-failed batches can be resumed efficiently, we need to design a system that allows us to track both the overall status of each batch and the individual row statuses. This involves using appropriate data structures and indexing strategies within SQL.

#### Triage

The core question is about designing an efficient storage mechanism for batch inference results in SQL. The interviewer might be probing into your knowledge of SQL, database design, and how to handle partial failures gracefully. We need to ensure that we can identify which rows have been processed and which haven't, allowing us to resume only the incomplete rows.

#### First-Principles Solution
Table Design: Create a table with columns for batch_id, input_row_id, and additional fields such as status (to track if the row has been processed), result (for storing the inference result), and possibly timestamp for when the operation was last attempted.
Indexing: Proper indexing is crucial to ensure that we can quickly look up which rows need processing. We'll create indexes on (batch_id, status) and (batch_id, input_row_id).
Handling Partial Failures: Implement a mechanism to mark rows as failed temporarily and then update them once the job resumes.
Concurrency Control: Ensure that updates do not interfere with each other by using appropriate transactions and locking mechanisms.
Detailed Solution

Table Schema:

SQL
CREATE TABLE batch_inference_results (
    batch_id INT NOT NULL,
    input_row_id INT NOT NULL,
    status VARCHAR(50) CHECK (status IN ('NEW', 'IN_PROGRESS', 'FAILED', 'COMPLETED')),
    result TEXT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (batch_id, input_row_id),
    INDEX idx_batch_status (batch_id, status)
);

Inserting New Rows: When a new batch is started, insert rows for all the input rows that need processing.

SQL
INSERT INTO batch_inference_results (batch_id, input_row_id, status)
SELECT :batch_id, row_id, 'NEW' FROM input_rows;

Processing Rows: To process rows in a batch, we can use a transactional approach to ensure atomicity and consistency.

SQL
START TRANSACTION;

UPDATE batch_inference_results SET status = 'IN_PROGRESS', timestamp = NOW()
WHERE batch_id = :batch_id AND input_row_id IN (:input_row_ids) AND status = 'NEW';

-- Assuming we have a function `process_row` that returns the result and updates the status
FOR row_id IN :input_row_ids DO
    SET @result = process_row(row_id, :batch_id);
    UPDATE batch_inference_results SET result = @result WHERE batch_id = :batch_id AND input_row_id = row_id;
    -- Optionally, set status to 'COMPLETED' if the processing is successful
END;

COMMIT;

Handling Partial Failures: If a failure occurs during processing, we can mark the rows as FAILED and attempt to resume later.

SQL
UPDATE batch_inference_results SET status = 'FAILED', timestamp = NOW()
WHERE batch_id = :batch_id AND input_row_id IN (:input_row_ids) AND status = 'IN_PROGRESS';

Resuming a Batch: To resume a partially-failed batch, we can select rows that need to be processed again.

SQL
SELECT input_row_id FROM batch_inference_results
WHERE batch_id = :batch_id AND status = 'FAILED';

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the key aspects are creating a robust table structure with proper indexing, handling partial failures gracefully, and ensuring that operations are atomic to avoid data inconsistencies. Common mistakes include not using transactions for critical updates or failing to properly handle concurrency issues.

If asked further, you might discuss how to scale this design as the number of batches grows, potential performance optimizations (like partitioning by batch_id), and how to manage large numbers of rows efficiently.

**Comparison:**

```
Column Name	Data Type	Constraints
batch_id	INT	NOT NULL
input_row_id	INT	NOT NULL
status	VARCHAR	CHECK (status IN ('NEW', 'IN_PROGRESS', 'FAILED', 'COMPLETED'))
result	TEXT
timestamp	DATETIME	DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
Column Name	Data Type	Constraints
row_id	INT	PRIMARY KEY
-- Other relevant columns
```

**Code:**

```sql
-- Table Schema
CREATE TABLE batch_inference_results (
    batch_id INT NOT NULL,
    input_row_id INT NOT NULL,
    status VARCHAR(50) CHECK (status IN ('NEW', 'IN_PROGRESS', 'FAILED', 'COMPLETED')),
    result TEXT,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (batch_id, input_row_id),
    INDEX idx_batch_status (batch_id, status)
);

-- Inserting New Rows
INSERT INTO batch_inference_results (batch_id, input_row_id, status)
SELECT :batch_id, row_id, 'NEW' FROM input_rows;

-- Processing Rows
START TRANSACTION;
UPDATE batch_inference_results SET status = 'IN_PROGRESS', timestamp = NOW()
WHERE batch_id = :batch_id AND input_row_id IN (:input_row_ids) AND status = 'NEW';

FOR row_id IN :input_row_ids DO
    SET @result = process_row(row_id, :batch_id);
    UPDATE batch_inference_results SET result = @result WHERE batch_id = :batch_id AND input_row_id = row_id;
END;

COMMIT;

-- Handling Partial Failures
UPDATE batch_inference_results SET status = 'FAILED', timestamp = NOW()
WHERE batch_id = :batch_id AND input_row_id IN (:input_row_ids) AND status = 'IN_PROGRESS';

-- Resuming a Batch
SELECT input_row_id FROM batch_inference_results
WHERE batch_id = :batch_id AND status = 'FAILED';
```

**Data:**

```sql
CREATE TABLE input_rows (
    row_id INT PRIMARY KEY,
    -- Other relevant columns
);
```

