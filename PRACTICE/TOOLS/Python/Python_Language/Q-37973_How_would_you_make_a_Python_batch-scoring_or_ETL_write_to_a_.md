---
type: "practice"
difficulty: "Deep"
---

# How would you make a Python batch-scoring or ETL write to a warehouse idempotent so re-running a failed job across millions of rows never duplicates data?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotent Event Handlers](<../../../../TOOLS/Python/Python_Language/Idempotent_Event_Handlers.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a Python-based batch-scoring or ETL process is idempotent, meaning it can be safely run multiple times without changing the result beyond the initial run, we need to handle potential duplicates when re-running failed jobs on millions of rows. The key idea here is to maintain an up-to-date record of processed rows and ensure that any row processed more than once is ignored.

The core steps involve:

Storing a unique identifier for each row in a database.
Checking the state of each row before processing.
Processing only those rows which have not been processed yet.
Marking the row as processed after successful write to ensure no reprocessing.

**Example input / output:**

```text
# Example usage
conn = psycopg2.connect("dbname=your_db user=your_user password=your_password")
cur = conn.cursor()

unprocessed_rows = get_unprocessed_rows(cur, "your_table_name")
for row_id in unprocessed_rows:
    process_row(row_id)
    mark_row_as_processed(cur, row_id)

conn.commit()
```

