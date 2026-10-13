---
type: "practice"
difficulty: "Deep"
---

# Your p99 query latency doubled but CPU and IO look flat. How would you investigate?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Percentiles vs Averages](<../../../../01_Databases_and_SQL/Database_Operations/Monitoring/Percentiles_vs_Averages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When your p99 query latency doubles while CPU and IO metrics remain stable, it suggests that the issue lies somewhere else in the system beyond traditional resource bottlenecks. This scenario points towards a potential issue like contention, network latency, or possibly an infrastructure change. To systematically investigate, we need to look at various aspects of the database environment, including query execution plans, slow query logs, and any recent changes.

To start, let's restate what the question is probing: it is asking for methods to diagnose performance issues in a database system that don’t align with typical resource constraints. The key assumptions here are that CPU and IO are not the primary bottlenecks, so we need to focus on other factors.

Step-by-Step Investigation

Review Slow Query Logs: First, check your slow query logs to identify which queries have become slower. Look for common patterns or specific queries that now take longer to execute.

Analyze Execution Plans: Use EXPLAIN statements or similar tools to review the execution plans of these identified queries. Check if there are any significant changes in how data is being retrieved, such as increased table scans instead of indexes.

Check for Contention: Contention can arise from various sources, including locking mechanisms, parallelism settings, and transactional bottlenecks.

Monitor lock waits using tools like SHOW ENGINE INNODB W_STATUS (MySQL) or equivalent commands in other databases.
Look at session wait statistics to see if there are any unusual spikes.

Network Latency: Given that CPU and IO metrics look flat, increased latency could be an indicator of network issues between the application layer and the database server.

Use tools like ping, traceroute, or packet analyzers (e.g., Wireshark) to diagnose any potential network problems.

Infrastructure Changes: Check if there have been recent changes in your infrastructure, such as new hardware, software updates, or configuration tweaks that might affect performance.

Review database and application configurations for any recent modifications.
Ensure that load balancers or proxy servers (if used) are not introducing additional latency.

Database Configuration Tuning: Sometimes, simple reconfigurations can drastically improve performance without needing major changes.

Adjust settings like innodb_buffer_pool_size, sort_buffer_size, and others in MySQL.

#### Cons
ider increasing the number of connections if the query is resource-intensive but underutilizing current resources.

Performance Metrics: Collect additional metrics to get a holistic view:

Use tools like Prometheus, Grafana, or custom scripts to gather more detailed performance data.
Look at cache hit rates, memory usage, and overall system health.

#### Closing Summary

In summary, the key approach is to systematically rule out resource bottlenecks by leveraging execution plans, slow query logs, and other diagnostic tools. This process helps identify whether the issue lies in query optimization, contention management, or possibly network latency. By addressing these areas, you can pinpoint the root cause of the increased p99 latency.

#### Common Mistakes/Follow-Ups
Ignoring Network Latency: Focusing too heavily on CPU and IO without considering network performance.
Overlooking Configuration Changes: Assuming that recent infrastructure changes are not significant enough to impact performance.
Not Collecting Detailed Metrics: Relying solely on high-level monitoring tools instead of gathering granular data for analysis.

**Comparison:**

| column1 | column2 |
|---------|---------|
| dataA   | dataB   |
| dataC   | dataD   |

**Code:**

```sql
-- Example: Review slow query logs in MySQL
SHOW PROCESSLIST;
SELECT * FROM information_schema.PROCESSLIST WHERE TIME > 2;

-- Analyze execution plans
EXPLAIN SELECT ... ;
```

**Example input / output:**

```text
Input:
- p99 query latency doubled but CPU and IO look flat.

Output:
- Investigate slow query logs.
- Analyze execution plans for significant changes.
- Monitor lock waits and session waits.
- Check network latency using tools like ping, traceroute.
- Review recent infrastructure and configuration changes.
```

**Data:**

```json
{
    "slowQueryLogs": [
        {
            "queryId": 1234,
            "startTime": "2023-10-05T14:00Z",
            "endTime": "2023-10-06T14:00Z",
            "queriesExecuted": 12000
        }
    ],
    "lockWaits": [
        {
            "waitId": 9876,
            "waitsSinceRestart": 500,
            "averageWaitTime": 1000000 // in microseconds
        }
    ]
}
```

**Flow:**

1. Review Slow Query Logs
2. Identify Slower Queries
3. Analyze Execution Plans
4. Check Lock Waits and SessionWaits
5. Investigate Network Latency
6. Review Recent Changes

