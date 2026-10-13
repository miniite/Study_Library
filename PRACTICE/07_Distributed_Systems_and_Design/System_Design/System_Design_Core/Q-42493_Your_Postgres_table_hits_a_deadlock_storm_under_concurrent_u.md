---
type: "practice"
difficulty: "Deep"
---

# Your Postgres table hits a deadlock storm under concurrent updates to overlapping rows. How do you read the deadlock graph in the logs and reorder your write access to break the cycle?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Primary-replica Databases](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Clickstream_Counting_Pipeline/Primary-replica_Databases.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with deadlocks in a PostgreSQL database, understanding how to interpret deadlock graphs from the logs is crucial. A deadlock occurs when two or more transactions are waiting on locks held by each other, creating an infinite loop that can bring your system to a halt. By analyzing the deadlock graph and reordering write access, you can break this cycle and ensure smooth operation.

#### Triage
Understanding the Question

The question asks for a deep understanding of how to read deadlock graphs from PostgreSQL logs and then reorder write access to prevent deadlocks. The interviewer is probing your knowledge of database concurrency control mechanisms, logging, and practical strategies to resolve performance bottlenecks.

#### First-Principles Solution

Reading the Deadlock Graph:

Log Analysis: Enable detailed logging in PostgreSQL by setting log_lock_waits to true. This will log all lock waits, which can help identify deadlocks.
Interpreting Logs: Look for specific patterns in the logs indicating a deadlock. Each transaction will have a unique ID (XID), and you'll see messages like "waiting for share lock on relation" or "could not obtain exclusive lock within query timeout."

Deadlock Graph Structure:

A typical deadlock graph might look something like this:
Transaction 1
Lock on Row 1
Lock on Row 2
Lock on Row 3
Each node represents a transaction, and each directed edge indicates that one transaction is waiting for a lock held by another.

Reordering Write Access:

Identify Bottlenecks: Determine which transactions are causing the most contention. Look at the frequency of deadlocks involving certain rows or tables.
Transaction Scheduling: Prioritize transactions based on their criticality and impact. For example, prioritize read-heavy transactions over write-heavy ones if possible.
Row-Level Locking: Ensure that locks are held for as short a time as necessary. Use SELECT FOR UPDATE judiciously to minimize lock duration.
Detailed Explanation

Enabling Logging:

SQL
-- Add this line in your postgresql.conf file
log_lock_waits = 'true'

This setting will generate detailed logs every time a transaction waits for a lock, making it easier to spot potential deadlocks.

Analyzing the Deadlock Graph:

In the example graph above, Transaction 1 and Transaction D are in a deadlock because each is waiting on a lock held by the other.
By identifying the root cause (e.g., frequent locking of rows), you can address it more effectively.

Reordering Write Access:

For instance, if multiple transactions frequently update the same row, consider using optimistic locking or reducing the granularity of locks.
Alternatively, implement a write barrier where high-priority writes are scheduled before lower-priority ones to minimize contention.

#### Closing Summary

By systematically analyzing the deadlock graph and reordering access patterns, you can mitigate the risk of deadlocks. This approach involves enabling detailed logging, interpreting logs effectively, and strategically managing transactional behavior to reduce lock conflicts.

#### Common Mistakes/Follow-Ups
Misinterpreting Logs: Failing to correctly read or misinterpret the log messages can lead to incorrect conclusions.
Overkill Reordering: Making sweeping changes without a clear understanding of the underlying issues might introduce other problems.
Performance Trade-offs: Prioritizing certain transactions too aggressively could negatively impact overall system performance.

**Code:**

```sql
-- Example configuration for enabling detailed logging in postgresql.conf
log_lock_waits = 'true'
```

