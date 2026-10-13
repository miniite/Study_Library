---
type: "practice"
difficulty: "Intermediate"
---

# How do you validate that data landed in the correct partition after an ETL load, and what checks do you automate?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Concurrent Partition Overwrites](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Concurrent_Partition_Overwrites.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When validating data landing in the correct partitions during an ETL process, we need to ensure that the data has been correctly distributed based on the specified partitioning strategy. This involves verifying both the structural correctness and the business logic consistency of the data. For a Database Developer with intermediate experience, this means automating checks through validation scripts and leveraging database features like partitioning constraints.

To start, let's break down what we're probing: the need to validate that our ETL job has correctly placed records into their respective partitions. This involves understanding the partitioning scheme (e.g., range, list, or hash) and ensuring that each record adheres to this scheme. Additionally, we must check for any potential data integrity issues.

#### Triaging the Question

The core of the question is:

Validation Strategy: How can we ensure that records are placed correctly in partitions?
Automation: What checks should be automated as part of our ETL process?

#### Assumptions

We're using a relational database system with support for partitioning.
The ETL job has already been run, and we need to validate the results.

#### First-Principles Solution
Step 1: Understanding Partitioning Strategy

Partitioning strategies can vary:

Range Partitions: Based on a range of values (e.g., dates).
List Partitions: Based on explicit list values.
Hash Partitions: Based on hash functions.

For this example, let's assume we're using range partitioning by date and hash partitioning by an ID field.

Step 2: Creating Validation Scripts

We need to write validation scripts that check:

Partition Count: Ensure the number of partitions matches expectations.
Record Distribution: Check if records are distributed across partitions correctly.
Data Integrity: Verify no duplicate or out-of-range data exists.

Here’s a basic outline of how we might approach this:

SQL
-- Example Validation Script

-- Step 1: Count Partitions
SELECT COUNT(*) FROM information_schema.partitions WHERE table_name = 'your_table';

-- Step 2: Check Record Distribution (Assuming range partition by date)
WITH partition_distribution AS (
    SELECT
        partition_name,
        COUNT(*) as record_count
    FROM
        your_table
    GROUP BY
        partition_name
)
SELECT * FROM partition_distribution;

-- Step 3: Ensure No Out-of-Range Records
-- Example for a date range partition (assuming records must be between '2021-01-01' and '2022-12-31')
WITH out_of_range AS (
    SELECT
        id,
        created_at
    FROM
        your_table
    WHERE
        created_at < '2021-01-01' OR created_at > '2022-12-31'
)
SELECT * FROM out_of_range;
Step 4: Automating the Checks

To make these checks automatic, we can use a combination of scheduled tasks and continuous integration tools. For example:

Scheduled Jobs: Use cron jobs or database scheduler to run validation scripts periodically.
Continuous Integration Tools: Integrate validation into CI/CD pipelines to ensure every ETL job is validated immediately post-execution.

#### Closing Summary

In summary, validating data in the correct partitions involves understanding the partitioning strategy, writing targeted SQL queries to check partition counts and record distributions, and automating these checks through scheduled tasks or CI/CD integrations. Common mistakes include neglecting edge cases like out-of-range values or failing to account for all possible partition types.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking non-standard partition types (e.g., list partitions).
Mistake: Failing to consider data integrity checks, such as ensuring no duplicates.

#### Follow-Up
Discussing the trade-offs between different partitioning strategies and how they impact performance.

**Code:**

```sql
-- Example Validation Script

-- Step 1: Count Partitions
SELECT COUNT(*) FROM information_schema.partitions WHERE table_name = 'your_table';

-- Step 2: Check Record Distribution (Assuming range partition by date)
WITH partition_distribution AS (
    SELECT
        partition_name,
        COUNT(*) as record_count
    FROM
        your_table
    GROUP BY
        partition_name
)
SELECT * FROM partition_distribution;

-- Step 3: Ensure No Out-of-Range Records
-- Example for a date range partition (assuming records must be between '2021-01-01' and '2022-12-31')
WITH out_of_range AS (
    SELECT
        id,
        created_at
    FROM
        your_table
    WHERE
        created_at < '2021-01-01' OR created_at > '2022-12-31'
)
SELECT * FROM out_of_range;
```

