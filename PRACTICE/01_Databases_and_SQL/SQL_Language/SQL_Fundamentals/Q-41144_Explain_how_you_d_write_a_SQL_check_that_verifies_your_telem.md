---
type: "practice"
difficulty: "Deep"
---

# Explain how you'd write a SQL check that verifies your telemetry rollup is lossless by reconciling the summed raw sample count against the rollup's total count per bucket, and what a mismatch tells you.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Rollups](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure our telemetry system remains lossless, we need to verify that the aggregated (or "rollup") counts match the summed raw sample counts at regular intervals. This involves writing an SQL check that compares the total count from the rollup table against the sum of individual raw samples grouped by relevant buckets. If there's a mismatch, it indicates data loss or corruption somewhere in the pipeline.

#### Triaging the Question

The core of this question is about ensuring data integrity across different levels of aggregation in our telemetry system. The check involves reconciling two sets of counts: one from the rollup table and another computed from raw samples. We need to identify which part might be incorrect if there's a mismatch.

Key Assumptions:
Rollup Table: Contains aggregated data with columns like bucket_id, timestamp, and count.
Raw Sample Table: Contains individual raw data points with columns like timestamp, value, and possibly a bucket_id or similar identifier.
Regular Intervals: The check is run at specific intervals to ensure consistency over time.
Step-by-Step Solution

Identify Relevant Buckets and Time Frames:

Define the time frame for which we want to perform this reconciliation, e.g., an hour or a day.
Identify the buckets (e.g., different service instances) that need to be checked.

Compute Total Count from Raw Samples:

SQL
SELECT
    bucket_id,
    COUNT(*) AS raw_sample_count
FROM
    raw_samples
WHERE
    timestamp >= 'start_time'
    AND timestamp < 'end_time'
    AND bucket_id IN (SELECT id FROM buckets)
GROUP BY
    bucket_id;

Retrieve Total Count from Rollup Table:

SQL
SELECT
    bucket_id,
    SUM(count) AS rollup_count
FROM
    rollups
WHERE
    timestamp >= 'start_time'
    AND timestamp < 'end_time'
    AND bucket_id IN (SELECT id FROM buckets)
GROUP BY
    bucket_id;

Compare Counts and Identify Mismatches:

SQL
WITH raw_counts AS (
    SELECT
        bucket_id,
        COUNT(*) AS raw_sample_count
    FROM
        raw_samples
    WHERE
        timestamp >= 'start_time'
        AND timestamp < 'end_time'
        AND bucket_id IN (SELECT id FROM buckets)
    GROUP BY
        bucket_id
),
rollup_counts AS (
    SELECT
        bucket_id,
        SUM(count) AS rollup_count
    FROM
        rollups
    WHERE
        timestamp >= 'start_time'
        AND timestamp < 'end_time'
        AND bucket_id IN (SELECT id FROM buckets)
    GROUP BY
        bucket_id
)
SELECT
    bc.bucket_id,
    rc.raw_sample_count,
    rc.rollup_count
FROM
    rollup_counts AS rc
FULL JOIN
    raw_counts AS rc
ON
    rc.bucket_id = rc.bucket_id
WHERE
    COALESCE(rc.raw_sample_count, 0) != COALESCE(rc.rollup_count, 0);

#### Closing Summary
and Common Mistakes/Follow-Ups

This SQL query will compare the raw sample counts with the rollup counts for each bucket. If there's a mismatch, it means that either the raw samples are missing or aggregated incorrectly in the rollup table.

#### Common Mistakes

Forgetting to include all relevant buckets.
Overlooking time frames and filtering by timestamps.
Misinterpreting what a mismatch signifies (e.g., might be due to data ingestion delays rather than loss).

#### Follow-Ups
Data Ingestion Delays: Discuss how delays in ingesting raw samples can cause temporary mismatches.
Error Handling: How to handle and mitigate issues when discrepancies are found.
Monitoring and Alerts: Implementing real-time monitoring and alerts for such mismatches.

**Code:**

```sql
WITH raw_counts AS (
    SELECT
        bucket_id,
        COUNT(*) AS raw_sample_count
    FROM
        raw_samples
    WHERE
        timestamp >= 'start_time'
        AND timestamp < 'end_time'
        AND bucket_id IN (SELECT id FROM buckets)
    GROUP BY
        bucket_id
),
rollup_counts AS (
    SELECT
        bucket_id,
        SUM(count) AS rollup_count
    FROM
        rollups
    WHERE
        timestamp >= 'start_time'
        AND timestamp < 'end_time'
        AND bucket_id IN (SELECT id FROM buckets)
    GROUP BY
        bucket_id
)
SELECT
    bc.bucket_id,
    rc.raw_sample_count,
    rc.rollup_count
FROM
    rollup_counts AS rc
FULL JOIN
    raw_counts AS rc
ON
    rc.bucket_id = bc.bucket_id
WHERE
    COALESCE(rc.raw_sample_count, 0) != COALESCE(bc.rollup_count, 0);
```

**Example input / output:**

```sql
WITH raw_counts AS (
    SELECT
        bucket_id,
        COUNT(*) AS raw_sample_count
    FROM
        raw_samples
    WHERE
        timestamp >= '2023-10-01T15:00:00Z'
        AND timestamp < '2023-10-01T16:00:00Z'
        AND bucket_id IN (SELECT id FROM buckets)
    GROUP BY
        bucket_id
),
rollup_counts AS (
    SELECT
        bucket_id,
        SUM(count) AS rollup_count
    FROM
        rollups
    WHERE
        timestamp >= '2023-10-01T15:00:00Z'
        AND timestamp < '2023-10-01T16:00:00Z'
        AND bucket_id IN (SELECT id FROM buckets)
    GROUP BY
        bucket_id
)
SELECT
    bc.bucket_id,
    rc.raw_sample_count,
    rc.rollup_count
FROM
    rollup_counts AS rc
FULL JOIN
    raw_counts AS rc
ON
    rc.bucket_id = bc.bucket_id
WHERE
    COALESCE(rc.raw_sample_count, 0) != COALESCE(bc.rollup_count, 0);
bucket_id	raw_sample_count	rollup_count
1	10	10
2	20	20
```

**Data:**

```json
{
  "buckets": [
    {"id": 1, "name": "Bucket A"},
    {"id": 2, "name": "Bucket B"}
  ],
  "raw_samples": [
    {"timestamp": "2023-10-01T15:00:00Z", "value": 10, "bucket_id": 1},
    {"timestamp": "2023-10-01T16:00:00Z", "value": 20, "bucket_id": 2}
  ],
  "rollups": [
    {"timestamp": "2023-10-01T15:00:00Z", "count": 10, "bucket_id": 1},
    {"timestamp": "2023-10-01T16:00:00Z", "count": 20, "bucket_id": 2}
  ]
}
```

