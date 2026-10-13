---
type: "practice"
difficulty: "Deep"
---

# A replica keeps disconnecting with 'Got fatal error 1236 from master'. What are the common root causes and how do you fix each?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Failback After Failover](<../../../../01_Databases_and_SQL/Database_Operations/Replication/Failback_After_Failover/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

When a database replica (slave) repeatedly disconnects with the error "Got fatal error 1236 from master," it indicates that there's a critical issue in the replication process between the master and slave. This error typically arises due to problems such as network instability, hardware failures, or configuration misalignments. Understanding and addressing these issues is crucial for maintaining data consistency and reliability.

#### Triaging the Question

The question requires diagnosing common root causes of database replication errors specifically related to connection disruptions. The core aspects being probed are:

Understanding Replication Basics: Knowledge of how master-slave replication works.
Identifying Common Issues: Familiarity with common error codes and their implications.
Root Cause Analysis: Ability to systematically identify potential problems and devise corrective measures.

#### First-Principles Solution
1. Network Stability

Network instability can be a significant factor in causing the "Got fatal error 1236" issue. This code suggests that there are connection issues between the master and slave, possibly due to network congestion or packet loss.

Common Causes:

Poor network infrastructure (e.g., low bandwidth).
Network latency issues.
Firewalls or security groups misconfigured to block replication traffic.

Fixing the Issue:

Ensure the network connection between master and slave is stable and robust. Use tools like ping and traceroute to diagnose packet loss and latency.
Increase network bandwidth if necessary.
Adjust firewall rules or security group settings to allow replication traffic (usually on ports 3306 for MySQL).
2. Master-Slave Configuration Mismatch

Configuration mismatches between the master and slave can lead to connection issues. This is often seen in misconfigured binary log formats, GTIDs (Global Transaction Identifiers), or inconsistent time settings.

Common Causes:

Incompatible binlog formats on both servers.
Outdated replication slaves not properly handling new events from the master.
Misaligned server clocks leading to timing issues.

Fixing the Issue:

Verify and synchronize configuration parameters such as server-id, binlog_format, and gtid_mode between master and slave. Ensure they are consistent.
Use tools like mysqlbinlog or pt-show-ddl to compare schema differences and apply necessary changes on the slave.
3. Hardware and Resource Constraints

Insufficient resources (CPU, memory) on either the master or slave can also lead to replication failures. These constraints might cause one side to drop connections due to resource exhaustion.

Common Causes:

High CPU usage leading to delayed writes or packet queuing.
Insufficient memory causing swap space use and performance degradation.

Fixing the Issue:

Monitor system metrics using tools like top, htop, or iostat to identify resource bottlenecks.
Scale up hardware resources if necessary, or optimize queries and indexing to reduce load on both master and slave.
4. MySQL Specific Replication Issues

For MySQL-based setups, specific replication errors can be indicative of deeper issues:

Common Causes:

Outdated version of MySQL/MariaDB which doesn’t fully support certain features.
Incompatible DDL statements causing replication failures.

Fixing the Issue:

Update both master and slave to ensure they are running compatible versions.
Review recent schema changes and apply them incrementally or use mysqlbinlog to replicate specific events.

#### Closing Summary

In summary, addressing "Got fatal error 1236" requires a systematic approach of diagnosing network stability, configuration mismatches, hardware constraints, and version compatibility. Each root cause needs careful identification and targeted resolution using appropriate tools and monitoring mechanisms.

#### Common Mistakes
and Follow-Ups
Mistake: Assuming the issue is always related to network instability without considering other factors like configuration settings.

#### Follow-Up
Questions:
How do you ensure data integrity during schema changes in a master-slave setup?
What steps would you take if the slave consistently lags behind the master despite optimizations?

**Code:**

```sql
-- Example of checking replication status on MySQL/MariaDB
SHOW SLAVE STATUS \G;
```

**Example input / output:**

```text
# Example output from SHOW SLAVE STATUS
Slave_IO_Running: No
Slave_SQL_Running: Yes
Last_Error: Got fatal error 1236 from master
Error_Code: 1236
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. No
7. Yes
8. No
9. Identify Connection Issues
10. Network Stability?
11. Check Network Configuration
12. Misconfigured Parameters?
13. Verify and Sync Configurations
14. Hardware Constraints?
15. Monitor Resources; Optimizeor Scale Up
16. MySQL VersionIncompatibility?
17. Update Software Versions
18. Check Recent Schema Changes

