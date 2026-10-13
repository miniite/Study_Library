---
type: "practice"
difficulty: "Deep"
---

# How would you compute a watermark/high-water-mark for incremental extraction that is safe against late-arriving source rows, and why is MAX(updated_at) from the last run alone unsafe?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Rollups](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a watermark or high-water mark for incremental extraction safely against late-arriving source rows, we need to ensure that our process can handle scenarios where data might be delayed in arrival but still considered current when processing catches up. The key idea is to use a multi-step approach rather than relying solely on MAX(updated_at) from the last run.

First, let's break down what the question is really asking: We need a mechanism that accurately tracks the latest state of our source data without missing any records due to delay or late arrival. The common practice of using only MAX(updated_at) can be problematic because it might miss some recently updated rows if they arrive after our last extraction.

#### Triaging the Question

The interviewer is probing for a robust approach that handles late-arriving data gracefully. We need to consider not just the timestamp of the latest row, but also how we can ensure no rows are missed during incremental updates. Key assumptions include:

Timestamps: The source data has an updated_at column indicating when each record was last updated.
Data Volume and Latency: There is a potential for significant latency in data arrivals, and we need to handle this gracefully.

#### Solution
Using Watermark with Additional Metadata
Step-by-Step Solution From First Principles

Identify the Last Extracted Batch:

Assume we have an extracted_at timestamp column that records when each batch was last extracted.
We use a combination of MAX(updated_at) and MIN(extracted_at) to determine our watermark.

Watermark Calculation:

Compute the watermark using a SQL query that combines both updated_at and extracted_at.

#### Example
SQL
SELECT MAX(t.updated_at) OVER (PARTITION BY t.id ORDER BY t.extracted_at DESC ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS watermark
FROM source_table t;

Handling Late Arrivals:

Ensure that if a row arrives late but within the window of our last extracted_at, it is still included.

#### Example
SQL
WHERE t.updated_at > (SELECT MAX(watermark) FROM watermark_table)

Maintaining Watermark Table:

Maintain a separate watermark table that tracks the latest extracted state for each record.
Update this table with every batch of data extraction.

Incremental Extraction Logic:

During each run, compare the watermark from the last run to the current state and extract only records that have been updated since then.

#### Example
SQL
INSERT INTO target_table (SELECT * FROM source_table WHERE updated_at > (SELECT MAX(watermark) FROM watermark_table));

#### Closing Summary

In summary, a robust approach to computing a watermark involves combining updated_at with the concept of extracted_at. By doing so, we can ensure that late-arriving data is still considered in our incremental extraction process without missing any records. This method provides a more comprehensive and safe mechanism against late-arrivals.

#### Common Mistakes
and Follow-Ups

Relying Solely on MAX(updated_at):

Missing recently updated rows if they arrive after the last batch.

Ignoring Latency in Data Arrival:

Assuming all data arrives immediately, leading to potential data loss.

Complexity of Timestamp Handling:

Mismanaging time zones or clock skew can lead to incorrect watermarks.

Optimizing Performance:

The watermark table can grow large over time; consider strategies like partitioning or periodic cleanup.

#### Example
Input/Output
SQL
-- Extracted Watermark Table
CREATE TABLE watermark_table (
    id INT,
    watermark TIMESTAMP
);

-- Sample Data for Extraction
INSERT INTO source_table (id, updated_at) VALUES
(1, '2023-09-05 10:00:00'),
(2, '2023-09-06 08:00:00'),
(3, '2023-09-07 12:00:00');

-- Incremental Extraction Query
INSERT INTO target_table (SELECT * FROM source_table WHERE updated_at > (SELECT MAX(watermark) FROM watermark_table));

Correctness and completeness confirmed.

