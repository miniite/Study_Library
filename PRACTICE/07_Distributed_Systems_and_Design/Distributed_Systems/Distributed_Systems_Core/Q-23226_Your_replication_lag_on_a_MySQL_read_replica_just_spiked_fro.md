---
type: "practice"
difficulty: "Deep"
---

# Your replication lag on a MySQL read replica just spiked from 50ms to 40s during peak traffic. Walk me through your debugging steps and the top three root causes you'd check first.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Partition Lag and Hot Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with an unexpected spike in replication lag on a MySQL read replica from 50ms to 40s during peak traffic, the immediate concern is ensuring data consistency and identifying potential bottlenecks. The key idea here is that we need to methodically debug both the primary (master) and replica nodes, while considering common issues like network latency, I/O performance, and configuration settings.

Triaging the Problem

First, it's crucial to understand what the replication lag represents: the time difference between changes made in the master database and when these changes are reflected on the read replica. A sudden increase during peak traffic indicates a significant issue that could impact performance and data consistency.

Key Assumptions:
Network Latency: The network between the master and the replica is stable.
Disk I/O Performance: Both the master and the replica have adequate disk I/O resources.
Configuration Consistency: The replication configuration on both nodes hasn’t changed recently.
Debugging Steps

The first step in debugging such an issue involves verifying the health of both the master and the replica:

Check Network Latency:

Use ping or tracert to measure network latency between the master and the replica.
Ensure there are no network issues that could be causing packet loss.

Monitor Disk I/O Performance:

Check for any disk I/O bottlenecks on both nodes using tools like iostat or fio.
Look for signs of high read/write operations, which can slow down replication.

Examine the Master’s Binary Log and Replica’s Relay Log:

Analyze the binary logs on the master to see if there are any large transactions that might be causing delays.
Check the relay logs on the replica to ensure they are processing these transactions correctly without any errors or slowdowns.

Review Replication Configuration:

Verify that the replication settings like binlog_format and replicate_do_db (if applicable) are configured appropriately.
Ensure there are no slow queries or long-running transactions on the master that could be impacting replication speed.

Check for Lock Contention:

Look into whether any tables or rows might be experiencing excessive locking, which can delay transaction completion and thus replication.

Analyze Replication Threads:

Use SHOW PROCESSLIST to check if there are any replication threads ( Slave I/O/SQL thread) that are hanging or not progressing as expected.
Monitor the Seconds_Behind_Master value, which should decrease over time; a sudden increase indicates an issue.
Top Three Root Causes

Based on the above checks, here are the top three root causes to investigate:

High Disk I/O Operations:

If the replica is experiencing high disk I/O operations during peak traffic, it might not be able to keep up with the master’s write rate. This can lead to significant replication lag.

Network Issues:

Latency spikes in the network between the master and replica could cause delays in receiving transactions from the master, leading to increased replication time.

Large Transactions or Lock Contention:

Large transactions on the master that take a long time to complete can back up the binary log. If these transactions are replicated without interruption, they will significantly increase the lag on the replica.
Lock contention on either node can cause delays in committing changes, which can be magnified during high traffic periods.

#### Closing Summary
and Common Mistakes

By following the above steps, we can systematically identify and address the root causes of increased replication lag. It's important to monitor these factors continuously to ensure they don't become a recurring issue. Common mistakes include overlooking network issues, assuming that I/O performance is constant across peak traffic periods, or failing to review large transactions and lock contention.

**Comparison:**

| Metric        | Value       |
|---------------|-------------|
| Seconds Behind Master | 40s         |
| Binary Log Size   | 2GB         |
| Relay Log Size    | 1.5GB       |

**Code:**

```
MERMAID
Copy
graph TD;
    A[Verify Network Latency] --> B[Check Disk I/O Performance];
    C[Monitor Replication Threads] --> D[Review Configuration Settings];
    E[Analyze Binary Logs and Relay Logs] --> F[Investigate Lock Contention];
```

**Example input / output:**

| Operation | Input | Output |
|-----------|-------|--------|
| Ping      | -     | 50ms   |
| Disk I/O  | Read  | 1000 IOPS/s |
|           | Write | 500 IOPS/s |

**Data:**

```json
{
  "networkLatency": {
    "ping": "50ms",
    "traceroute": "All paths have low latency"
  },
  "diskIOPerformance": {
    "readOps": "1000 IOPS/s",
    "writeOps": "500 IOPS/s"
  }
}
```

**Flow:**

1. Verify Network Latency
2. Check Disk I/O Performance
3. Maintain Disk IOPs
4. Monitor Replication Threads
5. Review Configuration Settings
6. Analyze Binary Logs and RelayLogs
7. Investigate Lock Contention

