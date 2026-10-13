---
type: "practice"
difficulty: "Deep"
---

# Your nightly query reads a table that another job is actively appending to and you get slightly different results run-to-run. Explain snapshot isolation on a lakehouse table and how pinning a snapshot id fixes non-reproducibility.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Optimistic Concurrency and Clocks](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/Optimistic_Concurrency_and_Clocks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where nightly queries reading from a delta table (common in data lakes) yield slightly different results due to concurrent appends, we need to understand and implement snapshot isolation properly. Snapshot isolation is crucial because it ensures that readers see a consistent view of the data at a specific point in time, which is essential for reproducibility.

Snapshot isolation works by creating a virtual copy (or snapshot) of the data at the moment a transaction starts. This snapshot remains unchanged throughout the transaction, ensuring that the query sees only one version of the data, no matter how the underlying data changes during its execution.

Triaging the Problem

The core issue here is non-reproducibility due to concurrent writes. In our context, the nightly query reads from a delta table where another job is appending new rows continuously. The problem arises because each read does not see the same state of the table, leading to slightly different results run-to-run.

Key Assumptions
Delta Lake: We are using Delta Lake for data storage.
Concurrent Appends: Another nightly job frequently appends new rows into the delta table.
Query Variability: The query results vary due to inconsistent state across reads.

#### First-Principles Solution

To fix this, we can pin a snapshot ID in our query. This means that instead of reading the latest state of the data (which is constantly changing), we read from a specific historical version of the data at the time when the transaction started.

Step-by-Step Explanation
Understanding Delta Lake: Delta Lake maintains ACID properties by creating a log for every operation and applying these operations in order to maintain a consistent state. Each write operation gets assigned a unique transaction ID, which helps in tracking changes over time.
Transaction Management: When you initiate a read query, it starts a new transaction. By default, this transaction sees the latest state of the data (which is not what we want here).
Pinning a Snapshot:
Use the USE SNAPSHOT command to specify a snapshot ID or timestamp.
For example, if you know that the nightly job appends rows at 10 PM every night, you can use the snapshot taken just before 10 PM as your read start point.

#### Example
Implementation
SQL
-- Assuming we have the latest snapshot ID from the previous run is 'snapshot_id_2345'
USE SNAPSHOT snapshot_id_2345;

SELECT * FROM my_table;

#### Closing Summary
and Common Mistakes/Follow-Ups

By using a specific snapshot, you ensure that your read operations see consistent data, thus fixing non-reproducibility. However, there are pitfalls to watch out for:

Snapshot Selection: Always select the correct snapshot ID or timestamp. If you choose one too early, it might miss recent changes.
Log Size and Performance: Over time, snapshots can accumulate in your logs, potentially increasing storage costs and query performance overhead.
Concurrency Issues: Ensure that other processes do not interfere with the snapshot you are using.

In conclusion, leveraging snapshot isolation through the USE SNAPSHOT command is a robust solution to ensure reproducibility when reading from delta tables in a distributed system environment.

**Code:**

```sql
-- Example SQL to pin a specific snapshot
USE SNAPSHOT snapshot_id_2345;
SELECT * FROM my_table;
```

