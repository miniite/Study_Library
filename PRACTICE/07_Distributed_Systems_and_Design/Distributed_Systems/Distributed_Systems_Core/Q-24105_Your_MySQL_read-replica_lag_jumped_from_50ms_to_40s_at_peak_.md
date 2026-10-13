---
type: "practice"
difficulty: "Deep"
---

# Your MySQL read-replica lag jumped from 50ms to 40s at peak. Walk me through your debugging: single-threaded replication apply, long-running transactions on the primary, big batch writes, and how you'd confirm the root cause from the replica's status.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Partition Lag and Hot Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a read-replica’s lag suddenly increases, it can be quite alarming, especially if the change is significant (from 50ms to 40s). In this scenario, as a Database Developer with an Architect/Principal level of expertise, I would systematically debug the issue by focusing on key areas: single-threaded replication apply, long-running transactions on the primary, and big batch writes. The goal is to confirm which factor or combination thereof is causing the lag.

Firstly, let's triage the situation:

Understanding the Question: The question probes our ability to identify performance bottlenecks in a distributed system, specifically focusing on replication mechanisms.
Key Assumptions:
We are dealing with a MySQL setup where read-replica is used for read traffic offloading.
The primary database is experiencing some form of increased latency or transactional throughput that affects the replica's ability to catch up.

To debug this, we would start by gathering detailed information from both the primary and replica databases. Here’s how I would approach it:

Step 1: Analyze Replication Status

Firstly, check the replication status on the read-replica using MySQL commands:

SQL
SHOW SLAVE STATUS\G

This command provides essential metrics like Slave_IO_Running, Seconds_Behind_Master, and Last_Error. If Seconds_Behind_Master shows a significant increase or if Slave_IO_Running indicates an issue, we need to investigate further.

Step 2: Examine Primary Database

Next, examine the primary database for any signs of long-running transactions or high write throughput:

SQL
SHOW FULL PROCESSLIST;

This command lists all current connections and their states. Look for queries that have been running for a long time (over 5 seconds would be suspicious) as they could block replication threads.

Step 3: Monitor Batch Writes

Large batch writes can significantly impact the replica’s ability to catch up:

SQL
SELECT COUNT(*) FROM information_schema.INNODB_LOCK_WAITS;

This query checks for locks that might indicate a bottleneck. Additionally, monitor the innodb_log_wait_for_flush_bytes variable and ensure it doesn't spike.

Step 4: Review InnoDB Settings

Check if any InnoDB settings are misconfigured:

SQL
SHOW VARIABLES LIKE 'innodb_%';

Ensure that parameters like innodb_flush_log_at_trx_commit, innodb_buffer_pool_size, and innodb_log_file_size are set appropriately to minimize the risk of replication lag.

Step 5: Analyze Long-Running Queries

Identify any long-running queries or large transactions on the primary:

SQL
SELECT * FROM information_schema.INNODB_TRX;

This query shows active transactions. Look for transactions that have been running for a long time, especially those involving many rows.

Step 6: Confirm the Root Cause

Once we gather all this information, we can confirm the root cause:

Single-threaded Replication Apply: If there are no long-running queries or large batch writes but Seconds_Behind_Master is still high, it might indicate a single-threaded apply issue.
Long-Running Transactions: Large transactions or queries blocking replication threads will directly impact the replica's ability to catch up. Ensuring that all critical operations are optimized and avoiding holding locks for extended periods can help mitigate this.
Big Batch Writes: High frequency of large batch writes could be overwhelming the system, especially if not properly balanced between primary and replicas.

#### Closing Summary

By methodically analyzing the replication status, examining the primary database transactions, monitoring batch writes, reviewing InnoDB settings, and identifying long-running queries, we can pinpoint the exact cause of the increased lag. Ensuring that all operations are well-optimized and avoiding bottlenecks is key to maintaining consistent performance in a distributed system.

#### Common Mistakes
Overlooking Long-Running Transactions: Failing to identify and address transactions holding locks for extended periods.
Misconfigurations in InnoDB Settings: Not optimizing parameters that can affect replication speed.
Ignoring Batch Writes: Underestimating the impact of large batch writes on the system.

**Code:**

```sql
-- MySQL commands for debugging
SHOW SLAVE STATUS\G;
SHOW FULL PROCESSLIST;
SELECT COUNT(*) FROM information_schema.INNODB_LOCK_WAITS;
SHOW VARIABLES LIKE 'innodb_%';
SELECT * FROM information_schema.INNODB_TRX;
```

