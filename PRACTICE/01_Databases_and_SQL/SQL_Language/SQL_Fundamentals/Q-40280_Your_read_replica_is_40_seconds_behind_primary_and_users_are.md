---
type: "practice"
difficulty: "Deep"
---

# Your read replica is 40 seconds behind primary and users are complaining they don't see their just-saved data. How do you diagnose the lag source and what patterns fix read-after-write without abandoning replicas?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Replica Staleness](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Replica_Staleness.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a read replica lags 40 seconds behind the primary, it can significantly impact user experience, especially when they expect their recently saved data to be immediately available. To diagnose and address this issue, we need to identify the root cause of the lag, understand the system architecture, and explore potential solutions that balance performance with consistency.

TL;DR
Diagnose by checking network latency, database configuration settings (replication delay), transaction logs, and query execution times.
Patterns for fixing read-after-write issues include using event-driven architectures, optimistic concurrency control, or employing a cache layer to reduce the reliance on replicas.

#### Triaging the Question

The primary concern here is ensuring that users see their just-saved data promptly, which may not be possible with a 40-second lag due to the nature of eventual consistency in read-replica setups. The key points are:

Network Latency: Check if the replica server has higher network latency.
Configuration Settings: Examine database replication settings and configuration.
Query Execution: Analyze query execution times on both primary and replicas.

#### Assumptions
The application is built with a fullstack SDE role, handling both front-end and back-end responsibilities.
Read-write operations are balanced across the primary and replica databases.
Users are complaining about not seeing their just-saved data, implying a read-after-write scenario.

#### First-Principles Solution

Check Network Latency

Use tools like ping or traceroute to measure network latency between the primary database server and the replica servers.
High network latency could be causing delays in propagating write operations to the replicas.

Examine Replication Delay Settings

Check if there are any settings that delay replication, such as log_bin_trust_function_creators or rds.read_only (if using AWS RDS).
Ensure the binlog_format is set correctly; mixed or statement-based binlogs might not capture certain types of write operations.

Analyze Transaction Logs

Look at the MySQL binary logs to trace recent write operations and their replication status.
Use tools like mysqlbinlog to inspect log entries and identify any issues with transaction propagation.

Monitor Query Execution Times

Collect query execution times on both the primary and replica servers for recently executed write operations.
Compare these metrics to identify if certain queries are taking longer on one server than the other, indicating potential bottlenecks or misconfigurations.
Patterns for Fixing Read-After-Write Issues

Event-Driven Architecture

Implement an event-driven architecture where read requests are routed based on the latest available data.
Use a message queue (e.g., Kafka) to publish write operations and ensure they reach all replicas before being committed.

Optimistic Concurrency Control

Introduce optimistic concurrency control mechanisms in your application logic to handle potential conflicts gracefully.
For example, use version numbers or timestamps on entities to detect if a read operation has occurred after the last write.

Cache Layer

Introduce an in-memory cache layer (e.g., Redis) that stores recent writes and serves read requests from this cache.
This reduces the reliance on replicas for immediate reads, ensuring fresher data is available even with delayed replication.

#### Cons
istent Reads via Primary or Read-Only Instances

For critical reads, route them to the primary database instance to ensure they are served from a consistent state.
Use read-only instances if your database supports it (e.g., AWS Aurora) for non-critical reads that can tolerate some degree of lag.

#### Closing Summary

By diagnosing network latency, configuration settings, and query execution times, you can identify the root cause of the 40-second lag. Implementing event-driven architectures, optimistic concurrency control, or caching strategies will help mitigate read-after-write issues without abandoning replicas entirely.

#### Common Mistakes/Follow-Ups
Ignoring Network Latency: Assuming that only database configurations are to blame when network latency is a significant factor.
Over-relying on Caching: Not considering the additional complexity and potential data staleness issues introduced by cache layers.
Not Monitoring Properly: Failing to set up adequate monitoring and logging for write operations and their replication status.

**Comparison:**

| Setting Name              | Value       |
|---------------------------|-------------|
| read_only                 | false       |
| log_bin_trust_function_creators | true      |
| binlog_format             | ROW         |

**Code:**

```

-- Example SQL queries to check replication delay settings

SHOW VARIABLES LIKE 'read_only';

CODE
Copy

```bash
# Example command to ping replica server
ping <replica_ip_address>

# Example command to trace network latency
traceroute <primary_ip_address>
```

**Example input / output:**

```text
Input:
1. Query execution times on primary and replica for recent write operations.
2. Network latency measurements between primary and replicas.

Output:
1. Network latency is high (70ms).
2. Replication settings show that `log_bin_trust_function_creators` is set to true, which might not be capturing certain types of writes.

Diagnosis: High network latency and potential issues with binlog format.
```

**Data:**

```json
{
  "network_latency": {
    "primary_to_replica1": 50ms,
    "primary_to_replica2": 70ms
  },
  "replication_settings": [
    {"setting_name": "log_bin_trust_function_creators", "value": true},
    {"setting_name": "binlog_format", "value": "ROW"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Check Network Latency
4. High Latency?
5. Improve Network Connectivity
6. Examine Replication Settings
7. Set `binlog_format` toSTATEMENT if needed
8. Analyze Query Execution Times

