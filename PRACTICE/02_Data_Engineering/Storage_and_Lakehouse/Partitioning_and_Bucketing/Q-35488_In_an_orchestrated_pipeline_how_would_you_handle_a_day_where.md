---
type: "practice"
difficulty: "Intermediate"
---

# In an orchestrated pipeline, how would you handle a day where no data arrived for a partition?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Concurrent Partition Overwrites](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Concurrent_Partition_Overwrites.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When handling partitions in a data pipeline, missing data on a specific day can be managed by marking or logging such scenarios to ensure consistency and traceability. This involves creating a mechanism to detect null partitions, possibly filling them with placeholder data or ignoring them depending on the use case.

#### Triage

The question is essentially asking how to handle the situation where no new data arrives for a particular partition in an orchestrated pipeline. The interviewer might be probing your understanding of partitioning strategies and how to maintain robustness and reliability in data processing pipelines, especially when unexpected scenarios arise.

#### Assumptions

You are working with a big data framework like Apache Hadoop/Hive.
Data is stored in partitions based on date or another relevant timestamp.
Step-by-Step Solution

Identify the Issue: Determine which partition(s) did not receive any new data. This can be done by comparing the expected and actual counts of files/directories for a specific partition.

Log the Missing Data:

Log a message indicating that no data was received for this partition.

#### Cons
ider adding metadata to the log, such as the timestamp when the pipeline ran and the name of the missing partition.

Handle the Missing Partition:

Option 1: Fill with Placeholder Data
If it’s acceptable to have placeholder data in the database, use a predefined set of records (e.g., zero-filled entries) for that day.

#### Example
SQL command:
SQL
INSERT INTO table_name VALUES ('2023-04-01', 0, 0);
Option 2: Skip the Partition
If missing data is not acceptable and should be reflected in your database as a gap, skip processing that partition.

#### Example
command:
SQL
-- In Hive
INSERT INTO table_name PARTITION (date='2023-04-01') SELECT * FROM dummy_table;

Notify Stakeholders:

If the missing data is critical, notify relevant stakeholders (e.g., team leads, data analysts) about the issue.
Use a monitoring system to alert on such scenarios.

Review and Improve Pipeline Logic:

Review pipeline logic to ensure that unexpected missing data situations are handled gracefully.

#### Cons
ider adding retry mechanisms or backfill processes for missed partitions.

#### Closing Summary

Handling missing data in partitioned pipelines requires careful consideration of the impact on downstream systems and processes. By logging these events, filling with placeholder data where appropriate, or skipping problematic partitions, you can maintain the integrity and consistency of your data pipeline. Regularly reviewing and improving pipeline logic ensures robustness against unexpected issues.

#### Common Mistakes/Follow-Ups
Failing to Log Events: Missing logs make it hard to trace back issues.
Ignoring Critical Data Gaps: Filling gaps with placeholder data might mislead users if the system does not properly distinguish between real and placeholder data.
Overlooking Monitoring: Not setting up alerts can lead to unnoticed problems.

For follow-ups, the interviewer might ask about specific tools or strategies used for logging, handling placeholders, or improving pipeline reliability. Be prepared to discuss these aspects in detail.

**Code:**

```sql
-- Example of inserting placeholder data into a Hive table
INSERT INTO table_name VALUES ('2023-04-01', 0, 0);
```

