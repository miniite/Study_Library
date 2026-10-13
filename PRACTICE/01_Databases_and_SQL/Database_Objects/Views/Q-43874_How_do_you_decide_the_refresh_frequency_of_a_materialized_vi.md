---
type: "practice"
difficulty: "Deep"
---

# How do you decide the refresh frequency of a materialized view when the base data updates continuously but the report tolerates 15-minute lag? Walk through the SLA math.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized View Refresh Correctness](<../../../../01_Databases_and_SQL/Database_Objects/Views/Materialized_View_Refresh_Correctness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding the refresh frequency of a materialized view (MV) with continuous base data updates and acceptable report latency, we need to balance several factors including data freshness, consistency, system resources, and user expectations. The key is to ensure that the reports do not show outdated information while minimizing the overhead on the database.

The question asks for the SLA math of a 15-minute lag scenario. Let’s walk through the steps:

#### Triage

What the Question Is Asking:

We need to determine how often we should update the materialized view so that it reflects the most recent data but still allows for up to a 15-minute delay in report results.

#### Assumptions

The base tables are updated continuously.
Reports can tolerate a 15-minute latency, i.e., users will not notice differences if the MV is refreshed every 15 minutes or less.
We aim to balance between minimizing resource usage and ensuring data freshness for reports.

#### First Principles Solution

To achieve an acceptable refresh frequency with minimal impact on the system:

Determine Base Data Update Frequency:

Assume the base tables are updated once per minute, but sometimes updates can be more frequent or less frequent.

Define SLA Requirements:

Reports should not show data older than 15 minutes.

Calculate Refresh Interval:

If a materialized view is refreshed every T minutes and the base data changes every t seconds, we want to ensure that any report query executed within T + t/60 * 15 (15 minutes) shows only recent data.

SLA Math:

For example, if the system updates once per minute (t = 60), we can set T to be a multiple of this interval but within our acceptable lag time.
Detailed Steps

Step 1: Base Data Update Frequency Analysis:

Suppose base tables are updated every 30 seconds on average. This means that the MV needs to be refreshed at least once every 2 minutes (to account for potential delays).

Step 2: Refresh Interval Selection:

To ensure the report does not show data older than 15 minutes, we can set the refresh interval T such that it covers a significant portion of the update frequency. For instance, refreshing every 3 to 4 minutes would be a practical choice.

Step 3: Implementing the SLA:

If the base tables are updated once per minute (t = 60), then setting T = 3 or 4 minutes ensures that any report query within this time window will reflect recent data. This gives us a buffer to account for additional updates.

#### Closing Summary
and Common Mistakes

Summary:

By setting the refresh interval of the materialized view to be around 3 to 4 minutes, we can ensure reports do not show outdated information while keeping system overhead manageable.

#### Common Mistakes

Overly frequent refreshes can strain the database and increase latency, whereas infrequent refreshes may lead to stale data in reports. Balancing these factors is crucial.

#### Example
Calculation
PLAINTEXT
Base update frequency (t) = 30 seconds
Desired SLA (lag time) = 15 minutes (900 seconds)

Refresh interval T = 3 - 4 minutes
If a refresh occurs every 3.5 minutes, the system can handle occasional updates with minimal impact on report freshness.

#### Example Scenario
Suppose at t=60s, a new update happens and the MV is refreshed at t=210s (3.5 minutes). Any report query executed within this time frame will use the fresh data.

**Code:**

```sql
-- Example SQL for creating a materialized view with a 3-4 minute refresh interval
 CREATE MATERIALIZED VIEW sales_report AS
 SELECT * FROM sales_data;

 -- Example scheduled job to refresh MV every 3.5 minutes
 BEGIN;
   REFRESH MATERIALIZED VIEW CONCURRENTLY sales_report;
 END;
```

**Example input / output:**

```text
Input:
 - Base table `sales_data` is updated once per minute.

 Output:
 - Materialized view `sales_report` is refreshed every 3.5 minutes to ensure reports are within a 15-minute lag.
```

