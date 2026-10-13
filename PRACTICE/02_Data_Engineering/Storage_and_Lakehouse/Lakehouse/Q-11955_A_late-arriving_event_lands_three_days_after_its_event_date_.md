---
type: "practice"
difficulty: "Intermediate"
---

# A late-arriving event lands three days after its event date. How do you correctly update the affected daily partition in the gold table?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Lakehouse Fundamentals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Lakehouse_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When handling a late-arriving event, ensuring that it gets incorporated into the correct partition of your data lake or lakehouse is crucial to maintain data integrity and accuracy. For this scenario, we need to update the daily partition from three days ago in the gold table. Here's how you can approach this problem:

#### Triage
The question asks about processing a late-arriving event that landed three days after its actual event date. We need to ensure that such events are correctly updated in the appropriate daily partition of our data lake or lakehouse.

Key Idea: Late-arriving events require us to identify and update historical partitions without disrupting ongoing real-time processes.

#### First Principles Solution

Identify the correct partition for the event date.
Insert the late-arriving event into that partition.
Ensure no double-counting or data duplication issues.

Let's break this down step-by-step:

Step 1: Identify the Correct Partition

Assume we have a daily partitioned gold table structured as follows:

SQL
CREATE TABLE gold_table (
    event_date DATE,
    key INT,
    value STRING
)
PARTITIONED BY (event_date);

For an event that occurred on 2023-10-01 but arrived three days later on 2023-10-04, the correct partition would be event_date = 2023-10-01.

Step 2: Insert into the Correct Partition

To insert the late-arriving event, you can use a standard SQL INSERT statement:

SQL
INSERT INTO gold_table (event_date, key, value)
VALUES ('2023-10-01', 12345, 'late_arrival_value');
Step 3: Avoid Double-Counting

Ensure that the late-arriving event does not cause any double-counting in your analysis. You can achieve this by using a unique identifier and ensuring each event is processed once.

Step 4: Maintain Data Integrity

To maintain data integrity, you might want to implement additional checks or constraints. For example, you could use upsert logic (upserts are not natively supported in all databases but can be implemented via stored procedures or ETL pipelines):

SQL
MERGE INTO gold_table AS target
USING (
    SELECT '2023-10-01' as event_date, 12345 as key, 'late_arrival_value' as value
) AS source
ON (target.event_date = source.event_date AND target.key = source.key)
WHEN MATCHED THEN UPDATE SET value = source.value
WHEN NOT MATCHED THEN INSERT (event_date, key, value) VALUES (source.event_date, source.key, source.value);

#### Closing Summary
and Common Mistakes/Follow-Ups:

The correct approach involves identifying the appropriate partition based on the event date, inserting the late-arriving event into that partition, and ensuring no double-counting. Common mistakes include not updating the correct partition or failing to handle potential duplicates properly.

For follow-ups, you might want to discuss how this solution scales with more complex data models or real-time processing requirements.

**Code:**

```sql
-- Example SQL for inserting a late-arriving event into the correct partition
INSERT INTO gold_table (event_date, key, value)
VALUES ('2023-10-01', 12345, 'late_arrival_value');

-- Example upsert logic using MERGE in SQL
MERGE INTO gold_table AS target
USING (
    SELECT '2023-10-01' as event_date, 12345 as key, 'late_arrival_value' as value
) AS source
ON (target.event_date = source.event_date AND target.key = source.key)
WHEN MATCHED THEN UPDATE SET value = source.value
WHEN NOT MATCHED THEN INSERT (event_date, key, value) VALUES (source.event_date, source.key, source.value);
```

