---
type: "practice"
difficulty: "Expert"
---

# A stream reading from a Delta source fails with 'source table has been updated non-append'. What caused it and how do you recover safely?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Delta Live Tables and Serving](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Live_Tables_and_Serving.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a Databricks streaming job encounters the error "source table has been updated non-append," it typically means that the source Delta table has undergone an update operation (like UPDATE or MERGE) that invalidated the state of the streaming job. This can happen when data in the Delta table is modified in a way that the streaming query cannot handle, leading to conflicts.

To recover safely from this situation, we need to understand the core issue and implement robust recovery mechanisms. Let’s break down the problem step by step:

Understanding the Error:

The error indicates that the Delta Lake transaction log (which tracks all changes to the table) contains operations that conflict with the current state of the streaming job.
This can occur if, for example, a row was updated or deleted in the source table after it was last read by the streaming query.

#### Assumptions
and Context:

The Delta table is versioned, meaning previous states are stored in transaction logs.
The streaming job reads data from this Delta table using a snapshot mode that captures a consistent view of the data at the start of processing.
The non-append update could involve UPDATE, MERGE, or even DELETE operations.

#### First-Principles Solution

Identify the Issue: First, we need to understand which rows caused the failure by analyzing the transaction logs in the Delta table.
Handle Conflicts Gracefully: Implement mechanisms to handle these conflicts, such as reprocessing affected rows or ignoring them based on business logic.
State Management: Ensure that the streaming job maintains a consistent state and can recover from failures without losing data.

Detailed Steps:

Analyze Logs:
SQL
-- Query transaction logs to identify problematic transactions
SELECT * FROM delta.`<path_to_table>`._delta_log WHERE operation = 'update' OR operation = 'delete'
Reprocess Affected Rows:
Use a checkpoint mechanism in Databricks to save the state of the streaming job at regular intervals.
When an error occurs, restart the streaming job from the last known good checkpoint and reapply transactions that have not been affected by non-append updates.
Implement Retry Logic:
PYTHON
# Example retry logic using a custom function in Databricks notebook
def handle_retries(session):
    while True:
        try:
            result = session.readStream.format("delta").load("<table_path>").writeStream.outputMode("append").start()
            break
        except Exception as e:
            if "source table has been updated non-append" not in str(e):
                raise e  # Rethrow other exceptions
            print(f"Reprocessing failed rows due to {e}")
            time.sleep(30)  # Wait before retrying

handle_retries(spark)
Optimize Streaming Query:
Ensure that the streaming query is optimized to minimize the impact of non-append updates. For instance, using spark.conf.set("spark.sql.streaming.checkpointLocation", "<checkpoint_path>") can help manage state effectively.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

By understanding and handling the "non-append" update issue, we can ensure that our Databricks streaming jobs are robust against data modifications.

#### Common mistakes
include not properly managing checkpoints, failing to identify and handle conflicting transactions, and not implementing retry mechanisms effectively.

**Comparison:**

| Operation | Row ID  | Timestamp |
|-----------|---------|----------|
| update    | 12345   | 2023-10-01T14:30:00.000Z |
| delete    | 67890   | 2023-10-02T15:00:00.000Z |

**Code:**

```python
# Example retry logic in Python using Spark
def handle_retries(session):
    while True:
        try:
            result = session.readStream.format("delta").load("<table_path>").writeStream.outputMode("append").start()
            break
        except Exception as e:
            if "source table has been updated non-append" not in str(e):
                raise e  # Rethrow other exceptions
            print(f"Reprocessing failed rows due to {e}")
            time.sleep(30)  # Wait before retrying

handle_retries(spark)
```

**Example input / output:**

```text
Example input:

spark.readStream.format("delta").load("<table_path>").writeStream.outputMode("append").start()

Output (error):

org.apache.spark.sql.streaming.StreamingQueryException: source table has been updated non-append

Worked dry-run:
- Identify problematic transactions
- Restart streaming job from last known good checkpoint
- Reprocess affected rows
```

**Data:**

```sql
-- Example SQL query for transaction logs
SELECT * FROM delta.`<path_to_table>`._delta_log WHERE operation = 'update' OR operation = 'delete'
```

**Flow:**

1. Yes
2. No
3. Start
4. Is error 'non-append' update?
5. Identify problematictransactions
6. Save checkpoint state
7. Restart streaming job from lastknown good checkpoint
8. Reprocess affected rows
9. Continue processing normally

