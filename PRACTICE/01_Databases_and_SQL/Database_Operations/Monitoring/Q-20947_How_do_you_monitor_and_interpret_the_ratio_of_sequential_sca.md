---
type: "practice"
difficulty: "Deep"
---

# How do you monitor and interpret the ratio of sequential scans to index scans on key tables?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Database Health Signals](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Database_Health_Signals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure database performance and health, monitoring the ratio of sequential scans to index scans is crucial. This metric helps us understand how well our queries are leveraging indexes and whether we need to optimize our query plans or indexing strategy. In this scenario, a higher ratio of sequential scans indicates that many queries might be missing good indexes, which can lead to poor performance.

The key idea here is to balance the trade-offs between using more CPU and I/O resources for index maintenance versus reducing execution time by utilizing well-designed indexes. We will start by defining what these terms mean and then dive into how we would implement a solution from first principles.

**Code:**

```sql
-- Example SQL query to monitor sequential scans and index scans
SELECT
    COUNT(CASE WHEN operation = 'seqscan' THEN 1 END) AS seqscans,
    COUNT(CASE WHEN operation = 'indexscan' THEN 1 END) AS indexscans,
    COUNT(*) AS total_scans,
    (COUNT(CASE WHEN operation = 'seqscan' THEN 1 END) * 100.0 / COUNT(*)) AS seqscan_ratio
FROM pg_stat_statements
WHERE query LIKE '%key_table%';
```

**Example input / output:**

```text
Input:
SELECT
    COUNT(CASE WHEN operation = 'seqscan' THEN 1 END) AS seqscans,
    COUNT(CASE WHEN operation = 'indexscan' THEN 1 END) AS indexscans,
    COUNT(*) AS total_scans,
    (COUNT(CASE WHEN operation = 'seqscan' THEN 1 END) * 100.0 / COUNT(*)) AS seqscan_ratio
FROM pg_stat_statements
WHERE query LIKE '%key_table%';

Output:
+----------+------------+---------+---------------------+
| seqscans | indexscans | total_scans | seqscan_ratio       |
+----------+------------+---------+---------------------+
| 3425     | 1798       | 5223     | 65.50%              |
+----------+------------+---------+---------------------+
```

**Flow:**

1. Yes
2. No
3. Query Monitoring
4. Is the seqscan ratio high?
5. Investigate query performanceand indexing strategy
6. Monitor for a period to ensurestability
7. Optimize queries or addindexes
8. Maintain monitoring to avoidfuture issues

