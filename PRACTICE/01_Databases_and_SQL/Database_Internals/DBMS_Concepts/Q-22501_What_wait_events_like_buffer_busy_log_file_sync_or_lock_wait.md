---
type: "practice"
difficulty: "Expert"
---

# What wait events (like buffer busy, log file sync, or lock waits) would you look at first when a database is CPU-idle but queries are slow?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Batching Writes](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Batching_Writes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When investigating a scenario where a database is CPU-idle yet queries are still slow, the first step is to understand that the issue likely lies in waiting events or I/O bottlenecks rather than CPU utilization. The key wait events to look at initially are those that directly indicate resource contention or performance issues related to storage and network I/O.

**Comparison:**

| Event             | Total Waits | Time Waited (ms) | Waited Percentage |
|-------------------|-------------|-----------------|------------------|
| db file sequential read  |   1024         |    56320        |     67.89%       |
| log file sync            |    256         |    12800        |      15.34%      |
| buffer busy waits        |    128         |     6400        |      7.63%       |

**Code:**

```sql
-- Example SQL query to gather wait event statistics (assuming Oracle DBMS)
SELECT event, total_waits, time_waited, waited_percentage
FROM v$session_wait
WHERE event LIKE '%log%' OR event LIKE '%buffer%'
ORDER BY waited_percentage DESC;
```

**Example input / output:**

```text
Event | Total Waits | Time Waited (ms) | Waited Percentage
------|-------------|-----------------|------------------
db file sequential read  |   1024         |    56320        |     67.89%
log file sync            |    256         |    12800        |      15.34%
buffer busy waits        |    128         |     6400        |      7.63%
```

**Flow:**

1. Yes
2. No
3. Buffer busy waits
4. Log file sync
5. Identify CPU-idle, queries slow
6. Are wait events significant?
7. Focus on top wait events
8. Check for other factors likenetwork I/O, hardware limits
9. Investigate buffer cacheefficiency and fragmentation
10. Analyze transaction logoperations and size

