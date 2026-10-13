---
type: "practice"
difficulty: "Deep"
---

# How do you model the capacity headroom for connection storms and thundering herds after a dependency recovers or a cache flushes?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Capacity Planning and Throughput](<../../../../01_Databases_and_SQL/Database_Operations/Capacity_Planning/Capacity_Planning_and_Throughput.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model the capacity headroom for connection storms and thundering herds in a database environment, we need to focus on understanding the behavior of connections under high load conditions after a recovery event or a cache flush. The goal is to ensure that the database can handle sudden surges without running out of resources. Here’s a deep dive into how this can be approached:

First, let's understand what "connection storms" and "thundering herds" mean in this context:

Connection Storms: A rapid increase in connection requests over a short period.
Thundering Herds: When many clients request the same resource at once after a recovery or cache flush, leading to a high load on the database.

We need to consider several factors including the database's current capacity, the expected load, and any potential bottlenecks. The key is to model these events accurately so that we can predict the headroom needed for such scenarios.

**Comparison:**

```
State	Count
active	20
idle	30
State	New Count
total	60
active	30
idle	30
```

**Code:**

```sql
-- Example SQL for monitoring connection usage
SELECT
    COUNT(*) AS total_connections,
    SUM(CASE WHEN state = 'active' THEN 1 ELSE 0 END) AS active_connections,
    SUM(CASE WHEN state = 'idle' THEN 1 ELSE 0 END) AS idle_connections
FROM
    pg_stat_activity;
```

**Example input / output:**

```text
Input:
Total Connections = 50, Active Connections = 20, Idle Connections = 30

Output:
- If we need to handle a connection storm where 10 new connections are established suddenly,
- The new state will be Total Connections = 60, Active Connections = 30, Idle Connections = 30.
```

**Data:**

```text
- `total_connections`: Total number of current connections.
- `active_connections`: Number of currently active connections.
- `idle_connections`: Number of idle connections.
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Recovered
6. Recovered
7. Headroom Adequate
8. Headroom Insufficient
9. Initial State
10. Connection Storm?
11. Increase Active Connections
12. Cache Flush?
13. Increase Idle Connections toActive
14. Monitor and Adjust as Needed
15. Maintain Active Connections
16. Reduce Active Connections,Increase Idle
17. Continue Monitoring
18. Scale Up Resources orImplement Load Balancing

