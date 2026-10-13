---
type: "practice"
difficulty: "Expert"
---

# You are seeing write-write conflicts under SNAPSHOT isolation (error 3960). Explain the cause and how you would design around it.

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Read Committed Locking Anomalies](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Read_Committed_Locking_Anomalies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

SNAPSHOT isolation is designed to minimize locking by using a versioning technique, where each transaction sees the database state at the moment it started, rather than blocking on current locks. However, this approach can lead to write-write conflicts under certain conditions, as seen in error 3960. The cause of such conflicts lies in how the isolation level handles concurrent writes. In this answer, I'll explain the root cause and discuss potential solutions.

To start with, SNAPSHOT isolation works by taking a snapshot of the database state at the beginning of each transaction. This means that all read operations see a consistent version of the data as it was when the transaction started, without blocking other transactions from writing to the same data. However, if two transactions attempt to write to the same row, one after the other, both will see the original snapshot and proceed with their writes, leading to conflicts.

For example, consider a scenario where Transaction A updates row R1 at time T1, and Transaction B reads R1 at time T2 (just before A's update) but writes to it at T3. Both transactions would see the pre-update state of R1, which can lead to conflicting states in the database.

To design around these conflicts, we need to consider several strategies:

Increase Concurrency with Locking Strategies: Adjusting the locking strategy can help manage concurrency better. For instance, using row-level locks or increasing the isolation level temporarily can prevent write-write conflicts.
Optimistic Concurrency Control (OCC): Implement OCC mechanisms where each transaction checks for conflicting updates before committing changes. This involves adding a version number to data and comparing it upon commit.
Partitioning: If certain tables are prone to high concurrency, partition them into smaller, more manageable chunks. This can reduce contention and improve overall performance.
Revisiting the Schema Design: Sometimes, the schema design itself might be causing these conflicts. Reviewing and possibly redesigning the database schema can help mitigate issues.

In summary, write-write conflicts under SNAPSHOT isolation arise due to concurrent writes appearing as non-conflicting to each transaction because they see a snapshot of the state at the start. To resolve this issue, strategies such as adjusting locking mechanisms, implementing OCC, partitioning tables, or revisiting schema design can be effective.

**Code:**

```sql
-- Example Schema for demonstration (SQL Server)
CREATE TABLE Orders (
    OrderID int IDENTITY(1,1) PRIMARY KEY,
    ProductID int NOT NULL,
    Quantity int NOT NULL,
    LastUpdated datetime DEFAULT GETDATE()
);
```

