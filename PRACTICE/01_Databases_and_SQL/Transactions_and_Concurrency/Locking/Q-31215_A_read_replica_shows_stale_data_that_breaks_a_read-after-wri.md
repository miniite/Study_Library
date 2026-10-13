---
type: "practice"
difficulty: "Deep"
---

# A read replica shows stale data that breaks a read-after-write flow. How does replication lag interact with your concurrency assumptions and how do you fix it?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Range Locks and Retries](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Range_Locks_and_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

When dealing with replication lags in a database system, especially when using read replicas, stale data can break expected behavior like read-after-write consistency. This issue arises due to the asynchronous nature of replication and concurrency assumptions in your application. To address this, you need to understand how replication lag affects transactional consistency and implement strategies such as delayed writes, snapshot-based approaches, or optimistic concurrency control.

#### Triage

The question is probing your understanding of replication lag and its impact on read-after-write consistency, which is a critical aspect of database design and operation. We'll assume the primary goal is to maintain data integrity while managing the trade-offs between performance and consistency. The key assumptions are that there's an asynchronous replication setup (e.g., MySQL with GTID) and that your application relies heavily on write operations followed by read operations.

#### First-Principles Solution
Understanding Replication Lag

Replication lag occurs because a primary database server writes data to its own transaction log, and the replica(s) only apply these changes after a delay. This delay can be significant in high-latency networks or when the replica is under heavy load. As a result, read operations on the replica might return outdated data.

Concurrency Assumptions

In many database applications, there's an implicit assumption that writes and reads are consistent within the same transaction. However, with asynchronous replication, this can fail if the write happens before the replica has processed the necessary changes. This is a common issue in distributed systems where read replicas are used to scale read-heavy workloads.

Fixing Replication Lag

To mitigate stale data issues, you need to consider several strategies:

Delayed Writes: Ensure that critical writes (e.g., those involving user transactions) occur only after the replica has caught up. This can be achieved through various mechanisms like delayed transactions or by using a snapshot-based approach.

Snapshot-Based Approach: Take periodic snapshots of the primary database and apply them to the replicas. This ensures that all replicas are in sync at specific points in time, reducing the risk of stale data.

Optimistic Concurrency Control (OCC): Implement OCC strategies where you attempt to read and update the latest version of the data before committing a write. If another transaction has modified it since your last read, handle the conflict appropriately.

Replication Topology: Optimize the replication topology by reducing network latency and ensuring that replicas are as close as possible to their intended users.

#### Example Scenario

Suppose you have an e-commerce application where customers make purchases, which are recorded in the primary database. The purchase data is also needed on a read replica for analytics purposes. If a user makes a purchase and immediately tries to query the total sales from the replica, they might get outdated information if the replica hasn't caught up yet.

Detailed Steps
Identify Critical Operations: Determine which write operations are critical enough that their stale data could cause issues.
Monitor Replication Lag: Implement monitoring tools to track replication lag and alert when it exceeds a certain threshold.
Implement Delayed Writes or Snapshots: For high-impact writes, use delayed transactions or take regular snapshots and apply them to the replicas.

#### Closing Summary

By understanding and addressing replication lag, you can ensure that your application maintains the necessary level of consistency even with asynchronous data distribution. The key is to balance performance needs with transactional integrity by implementing appropriate strategies like delayed writes, snapshot-based approaches, or optimistic concurrency control.

#### Common Mistakes/Follow-Ups
Ignoring Replication Lag: Failing to account for replication lag can lead to inconsistent read-after-write scenarios.
Overly Aggressive Optimization: While optimizing performance is important, it should not compromise data integrity. Always keep a balance and monitor the system's behavior under different loads.

#### Follow-Up
Questions
How do you ensure that your delayed writes mechanism works effectively?
Can you explain how optimistic concurrency control can be implemented in this context?

**Comparison:**

```
Table Name	Columns
purchases	user_id, product_id, timestamp
```

**Code:**

```sql
-- Example of delayed write mechanism using MySQL transactions
START TRANSACTION;
-- Perform write operation on primary
INSERT INTO purchases (user_id, product_id) VALUES (?, ?);
COMMIT;

-- On replica side:
IF replication_lag > threshold THEN
  REPEAT UNTIL replication_lag <= threshold DO SLEEP(1); END REPEAT;
END IF;
```

**Example input / output:**

```text
Request:
- Primary writes a purchase transaction.
Response:
- Transaction committed on primary.

Replica:
- Initially shows old data due to lag.
- Synchronizes after some time.
```

**Data:**

```json
{
  "replication_status": {
    "primary": {
      "last_log_file_pos": "0x698745",
      "seconds_behind_master": 2
    },
    "replica": {
      "last_log_file_pos": "0x698745",
      "seconds_behind_master": 10
    }
  }
}
```

**Flow:**

1. No
2. Yes
3. Primary Write
4. Replication Log
5. Replica Sync?
6. Stale Data
7. Up-to-date Data

