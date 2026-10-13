---
type: "practice"
difficulty: "Deep"
---

# A dbt incremental model reads from a CDC-landed staging table but occasionally drops updates — walk me through how you'd design the merge/dedup logic to be correct under out-of-order events.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Feeding OLAP from OLTP](<../../../../02_Data_Engineering/Modeling_and_Warehousing/OLTP_vs_OLAP/Separating_OLTP_and_Analytics/Feeding_OLAP_from_OLTP.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of handling out-of-order events in a dbt incremental model that reads from a CDC-landed staging table, we need to design a robust merge and deduplication logic. The core challenge here is ensuring that updates are correctly applied even if they arrive out of order.

#### Triaging the Question

The question asks us to devise a strategy for handling out-of-order events in an incremental model using dbt (data build tool). Specifically, we need to ensure that updates to records in the staging table are properly merged into our target data model. The key aspects the interviewer might be probing include:

Understanding of CDC and its implications.
Knowledge of dbt's incremental models.
Experience with handling out-of-order events.

#### Assumptions
We have a CDC-landed staging table that contains updates to records, possibly in an unordered manner.
The target data model needs to be kept consistent with the latest state of each record.
Updates to a single record might arrive multiple times, but we only want the most recent one to take effect.

#### First Principles Solution

To handle out-of-order events correctly, we can leverage a combination of temporal logic and unique constraints within our dbt model. Here’s how we can achieve this:

Create a Unique Key: Ensure each record has a unique identifier (e.g., id) that can be used to track and deduplicate records.
Temporarily Store Updates in a Buffer Table: Use an intermediate buffer table to hold incoming updates before they are merged with the target model.
Merge Logic: Implement a merge operation that checks for existing records based on their unique key, updates them if necessary, and ignores duplicate updates.
Detailed Steps
Unique Key: Assign a unique identifier (id) to each record in the staging table. This ensures we can track which records have been processed.
Buffer Table: Create a buffer table that will temporarily hold new or updated records from the staging table.
SQL
CREATE TABLE buffer_table (
    id BIGINT,
    data JSONB,
    update_time TIMESTAMP
);
Merge Operation:
Insert new or updated records into the buffer table with their update_time.
Merge these updates into the target model using a merge statement that checks for existing records based on their unique key and update_time.
SQL
MERGE INTO target_table t
USING (
    SELECT id, data, update_time
    FROM buffer_table b
) s ON (t.id = s.id AND t.update_time < s.update_time)
WHEN MATCHED THEN
    UPDATE SET data = s.data;
Cleanup: After the merge operation, clear the buffer table to prepare for the next batch of updates.

#### Closing Summary
and Common Mistakes

This approach ensures that even if out-of-order events occur, only the latest update for each record will be applied to the target model. The key to success is maintaining a unique identifier and using temporal logic in the merge operation.

#### Common Mistakes

Failing to use a unique identifier can lead to duplicate updates being incorrectly merged.
Ignoring out-of-order events without proper handling can result in outdated data in the target model.
Self-Check

This solution covers all aspects of correctly handling out-of-order events in an incremental dbt model. The logic is robust and accounts for the necessity of maintaining the latest state of each record.

