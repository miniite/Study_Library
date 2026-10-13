---
type: "practice"
difficulty: "Deep"
---

# Your analytics team wants ad-hoc queries against production and it's causing lock contention and IO spikes. What isolation, replica, or CDC-based approach would you propose and why?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Isolation Anomalies](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Isolation_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of lock contention and I/O spikes from ad-hoc queries on a production database, we need to ensure that our database can handle these high-load scenarios without compromising performance or data integrity. We'll explore several strategies including isolation levels, replication, and CDC-based approaches.

#### Triage
Understanding the Problem

The key aspects of this problem are:

Lock Contention: Ad-hoc queries are causing locks on the database.
I/O Spikes: The queries are generating a significant amount of I/O operations, impacting overall performance.
Production Environment: We must avoid any changes that might affect the critical production systems.

#### First Principles Solution

We will leverage a combination of isolation levels and replication strategies to ensure high availability while minimizing contention.

Isolation Levels

Firstly, we should reassess the isolation level settings for our queries. By default, many databases use SERIALIZABLE or REPEATABLE READ isolation levels which can be overly restrictive and lead to excessive locking. We need to evaluate whether a lower isolation level such as READ COMMITTED or REPEATABLE READ could suffice while still ensuring data consistency.

Replication

Next, we'll implement replication strategies to distribute the read load away from the production database.

Read-Only Replicas: Create read-only replicas of the primary database. These replicas can handle ad-hoc queries without affecting the production environment.
Multi-Master Replication (optional): In some cases, if the application supports it, multi-master replication might be considered for write-heavy workloads, but this is less common.
CDC-based Approach

Change Data Capture (CDC) allows us to track changes in real-time and can help manage heavy read workloads more efficiently.

Materialized Views: Use materialized views that are periodically refreshed. This reduces the load on the primary database by caching the results of complex queries.
Triggers and Log Files: Implement triggers or directly access log files if supported to capture changes in real-time and apply them to the replicas.
Step-by-Step Solution

Assess Current Isolation Levels:

Identify which transactions are causing locks.
Evaluate whether a less restrictive isolation level (READ COMMITTED, REPEATABLE READ) can be used without compromising data integrity.

Implement Read-Only Replicas:

Set up read-only replicas using your database's replication capabilities (e.g., MySQL GTID-based replication, PostgreSQL streaming replication).
Ensure these replicas are kept up-to-date with the production system to handle ad-hoc queries efficiently.

#### Cons
ider Multi-Master Replication:

If the application supports it and the workload permits, consider implementing multi-master replication for write-heavy scenarios.
Monitor closely to avoid conflicts during writes.

Utilize Materialized Views or CDC:

Implement materialized views that are refreshed periodically to handle complex queries efficiently.
Alternatively, use CDC to capture changes in real-time and apply them to the replicas.

Monitor Performance:

Continuously monitor the performance of both the primary database and the replicas.
Tune parameters as necessary based on observed metrics (e.g., I/O wait times, lock waits).

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Common Mistakes
Overlooking the importance of isolation levels; failing to properly manage replication lag; not considering CDC for complex queries.

#### Follow-Ups
Exploring specific database features (e.g., MySQL Group Replication), understanding the impact on backup strategies, and discussing failover mechanisms.
Self-Check

This solution addresses the core issues effectively while maintaining data integrity and performance. All steps are carefully considered to minimize disruption in a production environment.

**Code:**

```sql
-- Example of adjusting isolation level in MySQL
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Example of creating a read-only replica using GTID-based replication
CHANGE MASTER TO MASTER_HOST='replica_host', MASTER_USER='replica_user',
MASTER_PASSWORD='password', MASTER_LOG_FILE='mysql-bin.000001',
MASTER_LOG_POS=468;
START SLAVE;
```

**Example input / output:**

```text
-- Example input-output for adjusting isolation levels
Input:
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

Output:
Transaction isolation level changed to READ COMMITTED.
```

**Data:**

```yaml
# Example of a materialized view configuration in PostgreSQL
CREATE MATERIALIZED VIEW sales_summary AS
SELECT customer_id, SUM(sales_amount) as total_sales
FROM sales
GROUP BY customer_id;
```

