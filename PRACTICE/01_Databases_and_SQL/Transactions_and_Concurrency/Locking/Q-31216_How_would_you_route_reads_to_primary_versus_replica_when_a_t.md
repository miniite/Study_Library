---
type: "practice"
difficulty: "Intermediate"
---

# How would you route reads to primary versus replica when a transaction just wrote data and must not read stale rows?

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Range Locks and Retries](<../../../../01_Databases_and_SQL/Transactions_and_Concurrency/Locking/Range_Locks_and_Retries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To route reads to the primary versus replicas in a scenario where a recent write transaction must ensure no stale data is read, we need to carefully consider the trade-offs between performance and consistency. The key idea here is leveraging a combination of state-of-read (SOR) techniques and database-specific configurations to achieve a balance that minimizes contention on the primary while ensuring eventual consistency across replicas.

#### Triage

The question is essentially asking how to manage read traffic in a distributed database setup where recent writes should not be visible during reads. The core aspects are:

Ensuring transactional integrity for recent writes.
Minimizing the latency of reads by routing them to a replica if possible.
Balancing the load on the primary server while maintaining consistency.

#### First Principles Solution
Understand Transactional Write: A write operation modifies data and is critical to ensure it reaches all necessary replicas before any read operations can see these changes.
Read Routing Strategy:
Primary Server: Used for writes and potentially critical reads that must be up-to-date immediately.
Replicas: Used for non-critical reads or when the primary is under heavy load to minimize latency.
Step-by-Step Solution
Identify Write Operations:
A transaction has just written data, meaning it will need to ensure all replicas have seen this write before any subsequent read operations.
Update Replicas:
The primary server should notify the replicas about the new write. This can be done through a replication protocol like asynchronous or synchronous replication.
Set State-of-Read (SOR) Marker:
Once all necessary replicas have acknowledged receipt of the write, set an SOR marker to indicate that any read operation within this transaction must honor the latest data up to this point.

#### Implementation
Details

Primary Server:

SQL
-- Example SQL for setting a commit flag on primary server
BEGIN TRANSACTION;
INSERT INTO table (data) VALUES ('new_data');
SET @commit_flag = 1; -- Mark transaction as needing read consistency

-- Notify replicas and wait until all acknowledgments are received
CALL notify_replicas('write', 'table_name', 'new_data');

COMMIT;

Replica Servers:

SQL
-- Example SQL for handling write notifications on replica servers
CREATE PROCEDURE handle_write_notification(IN operation VARCHAR(10), IN table_name VARCHAR(50), IN data JSON)
BEGIN
  IF operation = 'write' THEN
    UPDATE table_name SET ... WHERE ...
    -- Mark the write as acknowledged
    INSERT INTO ack_table (table_name, data) VALUES (table_name, data);
  END IF;
END;

Read Routing:

For a read operation that must be up-to-date with the latest writes, it should first check if the transaction has committed on all necessary replicas.
If not, route the read to the primary server; otherwise, allow routing to any replica.

#### Closing Summary

By using an SOR marker and ensuring writes are acknowledged by all relevant replicas before committing, we can ensure that reads see only up-to-date data. This approach balances consistency requirements with performance optimizations, allowing for efficient use of replicas while maintaining transactional integrity.

#### Common Mistakes/Follow-Ups
Failing to Acknowledge Writes: Not properly marking writes as acknowledged on replicas can lead to stale data being read.
Not Using SOR Correctly: Misusing or ignoring the SOR mechanism can result in inconsistencies during reads.
Overloading Primary Server: Failing to route non-critical reads to replicas can overload the primary, impacting overall performance.

