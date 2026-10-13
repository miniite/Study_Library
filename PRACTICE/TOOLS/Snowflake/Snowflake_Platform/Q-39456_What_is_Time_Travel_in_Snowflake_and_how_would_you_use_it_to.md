---
type: "practice"
difficulty: "Intermediate"
---

# What is Time Travel in Snowflake and how would you use it to recover accidentally deleted data?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Time Travel and Storage Costs](<../../../../TOOLS/Snowflake/Snowflake_Platform/Time_Travel_and_Storage_Costs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Time Travel in Snowflake is a feature that allows you to access historical states of tables and views within a specified time window, enabling recovery from accidental deletions or updates. This feature leverages Snowflake’s transaction log (Change Data Capture, CDC) to store changes made to the data over time.

#### Triage

The question is essentially asking for an explanation of Time Travel in Snowflake and how it can be used to recover deleted data. The interviewer might probe your understanding of when and why you would use this feature, as well as the limitations or best practices involved.

#### First-Principles Solution

#### Overview

What is Time Travel?: Time Travel in Snowflake allows users to query past states of tables and views using a point-in-time timestamp.
Why Use It?: Primarily for data recovery, auditing, or ensuring compliance with data governance policies.

How Does it Work?:

Transaction Log (CDC): Snowflake captures all data modifications in its transaction log, which is used to reconstruct the state of your tables and views at any point in time.
Query Syntax: You can use the AS OF TIMESTAMP clause in SQL queries to retrieve historical versions of data.

Using Time Travel for Data Recovery:

Identify the Problem: Determine the exact timestamp when the accidental deletion occurred.
Execute Query with AS OF: Run a query using the AS OF TIMESTAMP clause to access the data at that point in time. For example, SELECT * FROM my_table AS OF '2023-10-05T14:30:00'.

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: Time Travel is a powerful feature for recovering deleted data or auditing past states of your data in Snowflake. It requires careful management to ensure you can always access the historical data needed.

#### Common Mistakes
Not enabling Time Travel on tables that require it.
Overlooking the cost implications, as Time Travel increases storage costs and query processing time.
Failing to document when and why specific actions were taken with Time Travel.

#### Follow-Ups
The interviewer might ask about how you manage and optimize the use of Time Travel in a production environment.
They may also probe on best practices for enabling Time Travel and setting retention periods based on data sensitivity and business needs.

**Code:**

```sql
-- Example Query to Recover Deleted Data using AS OF TIMESTAMP
SELECT * FROM my_table AS OF '2023-10-05T14:30:00';
```

**Flow:**

1. Enable Time Travel on Table
2. Use AS OF TIMESTAMP Clausein Query
3. Retrieve Historical Data atSpecified Point-in-Time

