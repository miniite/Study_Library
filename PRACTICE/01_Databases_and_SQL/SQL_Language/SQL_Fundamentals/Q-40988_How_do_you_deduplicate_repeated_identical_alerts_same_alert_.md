---
type: "practice"
difficulty: "Intermediate"
---

# How do you deduplicate repeated identical alerts (same alert_name + host) keeping only the latest, using ROW_NUMBER?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Deduplication in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Deduplication_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To deduplicate repeated identical alerts while retaining the most recent ones, we can use the ROW_NUMBER() function in SQL to assign a unique row number based on specific criteria. The idea is to partition the data by alert_name and host, order it by some timestamp (e.g., alert timestamp), and then filter out all but the latest entry for each group.

Let's break down the solution step-by-step:

Understand the Question: The problem asks us to remove duplicate alerts that have the same alert_name and host. We need to keep only the most recent (latest) alert.

#### Assumptions
Each alert has a unique identifier, such as an alert_id.
There is a timestamp field (timestamp) that we can use to determine which row is more recent.

#### First-Principles Solution
Use the ROW_NUMBER() function within a window partitioning clause.
Partition by alert_name and host, order by timestamp in descending order (newest first).
Assign a row number to each alert based on its position within the ordered group.
Finally, filter out all rows except those with the lowest row number (ROW_NUMBER() = 1).

Here is an example SQL query that implements this logic:

SQL
WITH RankedAlerts AS (
    SELECT
        alert_id,
        alert_name,
        host,
        timestamp,
        ROW_NUMBER() OVER (PARTITION BY alert_name, host ORDER BY timestamp DESC) as rn
    FROM alerts
)
SELECT
    alert_id,
    alert_name,
    host,
    timestamp
FROM RankedAlerts
WHERE rn = 1;

#### Closing Summary
and Common Mistakes/Follow-Ups:
The ROW_NUMBER() function ensures that we have a unique row number for each combination of alert_name and host.
By ordering by the timestamp in descending order, we ensure that the most recent alert gets the lowest row number.
The common mistake is to use incorrect window functions or not properly partitioning the data. Always double-check your logic!

This approach is efficient and works well for most scenarios where you need to deduplicate alerts based on specific fields.

**Code:**

```sql
WITH RankedAlerts AS (
    SELECT
        alert_id,
        alert_name,
        host,
        timestamp,
        ROW_NUMBER() OVER (PARTITION BY alert_name, host ORDER BY timestamp DESC) as rn
    FROM alerts
)
SELECT
    alert_id,
    alert_name,
    host,
    timestamp
FROM RankedAlerts
WHERE rn = 1;
```

