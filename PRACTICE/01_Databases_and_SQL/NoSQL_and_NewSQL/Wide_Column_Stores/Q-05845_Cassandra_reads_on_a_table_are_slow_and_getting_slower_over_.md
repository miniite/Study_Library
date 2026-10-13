---
type: "practice"
difficulty: "Expert"
---

# Cassandra reads on a table are slow and getting slower over weeks. You suspect tombstones. How do you confirm it, and what causes tombstone buildup? Include gc_grace_seconds in your answer.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cassandra Replication and Read Path](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Cassandra_Replication_and_Read_Path.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To diagnose slow Cassandra reads due to tombstones, we need to understand the root cause and verify our suspicions by analyzing system metrics and configuration settings. Tombstones are markers indicating that a row or column family has been deleted but is still retained in storage for a period called gc_grace_seconds. Over time, as more deletions occur, these tombstones accumulate, leading to degraded read performance.

Triaging the Problem

The primary suspect here is gc_grace_seconds, which controls how long tombstones are kept. If this setting is too high or if there's an unusual pattern of deletion and write operations, it can lead to a buildup of tombstones. To confirm this, we need to:

Check the gc_grace_seconds Setting: Verify that gc_grace_seconds is set appropriately for your use case.
Analyze Read Latency Metrics: Look at read latency over time to see if there are spikes or trends that align with our hypothesis.
Review Write Operations: Examine write patterns and frequency to understand how many deletions are occurring.
Step-by-Step Solution from First Principles

Check gc_grace_seconds Configuration:

Cassandra's gc_grace_seconds default is 10800 (3 hours), but it can be configured higher depending on the use case.
Running a command like nodetool getconf gc_grace_seconds will show the current setting across all nodes.

Analyze Read Latency:

Use tools like nodetool cfstats to monitor read latency and other relevant metrics over time.
Look for increasing latencies that correlate with periods of high write activity, especially deletions.

Examine Write Patterns:

Analyze system.log files for tombstone-related messages (e.g., "Tombstone on row" logs).
Use nodetool compact to force a compaction and observe if there is an improvement in read performance.

#### Cons
ider enabling tracing with dse tracing enable to capture detailed execution paths during reads.

#### Closing Summary

Confirming tombstone buildup as the cause of slow reads involves a combination of configuration checks, latency analysis, and write pattern examination. Adjusting gc_grace_seconds can help manage tombstones more effectively, but it’s crucial to balance this setting with read performance requirements. Overly high values will lead to unnecessary storage costs and degraded read times.

#### Common Mistakes/Follow-Ups
Mistake: Ignoring the default value of gc_grace_seconds, assuming it is always set appropriately.

#### Follow-Up
Discussing how to optimize write patterns, such as using TTLs or batch updates for deletions, to reduce tombstone buildup.

**Comparison:**

| Column Family | Read Latency (95th Percentile) | GC Grace Seconds |
|---------------|-------------------------------|------------------|
| system         | 1.4 ms                        | 10800            |
| keyspace       | 5.6 ms                        | 2592000          |

**Code:**

```bash
# Check gc_grace_seconds setting
nodetool getconf gc_grace_seconds

# Monitor read latency over time
nodetool cfstats <keyspace>.<table>

# Analyze system logs for tombstone messages
grep -i "tombstone" /var/lib/cassandra/system.log
```

**Example input / output:**

```text
# Example input for nodetool cfstats command
Read latency stats:
95th percentile: 5.6 ms
Median: 1.4 ms

# Example output from nodetool compact
INFO  [main] 2023-10-05 12:00:00,000 Compaction completed for system.local (272ms)
```

**Data:**

```yaml
# Example YAML output from nodetool cfstats
{
  "keyspace": "system",
  "column_family": "schema_columnfamilies",
  "read_repair_chance": 1.0,
  "compaction": {
    "class": "SizeTieredCompactionStrategy",
    "min_threshold": 4,
    "max_threshold": 32
  },
  "gc_grace_seconds": 10800
}
```

**Flow:**

1. Yes
2. No
3. Check gc_grace_seconds
4. Verify current setting
5. Monitor read latency
6. Analyze write patterns
7. Adjust gc_grace_seconds
8. Force compaction
9. Monitor performanceimprovement

