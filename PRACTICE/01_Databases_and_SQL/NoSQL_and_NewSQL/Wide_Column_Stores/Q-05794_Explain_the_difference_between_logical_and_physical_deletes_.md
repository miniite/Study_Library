---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between logical and physical deletes in Cassandra, why a delete is itself a write (tombstone), and how a naive 'delete then reinsert' pattern can resurrect old data.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Cassandra Replication and Read Path](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Wide_Column_Stores/DynamoDB_Modeling_and_Capacity/Cassandra_Replication_and_Read_Path.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When comparing Cassandra to DynamoDB, one of the key differences lies in their handling of deletes—specifically, logical vs. physical deletes. In this explanation, I'll delve into how Cassandra manages deletions through tombstones and why a naive 'delete then reinsert' pattern can lead to data resurrection.

#### Triaging the Question

The question revolves around understanding delete operations in Cassandra, particularly focusing on the nature of these operations as writes (tombstones) and the potential pitfalls of a common deletion pattern. The interviewer is probing your knowledge of Cassandra's internal mechanisms for managing data deletion and the importance of correctly implementing delete operations to avoid unintended data retention.

#### First-Principles Solution

In Cassandra, a logical delete is not truly removing the row from storage; instead, it marks the row as deleted through a mechanism known as tombstones. These tombstones are essentially flags that tell the system to ignore reads for the specified range of columns or timestamps. The physical deletion of data only occurs during major compaction, which is a rare and resource-intensive operation.

Why Deletes Are Themselves Writes

A delete in Cassandra involves writing a tombstone entry into the storage system. This tombstone acts as a marker indicating that certain columns (or the entire row) are no longer valid for reads. Tombstones have a time-to-live (TTL), which determines how long they remain active before they expire and get cleaned up during compaction.

Naive 'Delete Then Reinsert' Pattern

A naive approach to deletion involves first performing a delete operation, followed by reinserting the row with updated data. This pattern can lead to data resurrection because Cassandra's internal mechanisms might not immediately mark all columns as tombstoned. During the reinsertion, some columns may still be present and accessible during reads.

Full Solution
Logical Deletes in Cassandra

Cassandra handles deletions by writing tombstones rather than physically removing rows. When a row is deleted, a tombstone with a TTL (Time To Live) is inserted into the system. This tombstone acts as a flag that marks the columns or entire row as invalid for reads. The duration of the tombstone's lifecycle depends on the specified TTL value.

Physical Deletion

Physical deletion in Cassandra only occurs during compaction, which is an operation designed to reclaim space and optimize storage by merging SSTables (Sorted String Tables). Compaction can take a long time and requires significant resources, making it less frequent compared to regular read and write operations.

Tombstone Time-to-Live

Tombstones have a TTL that determines how long they remain active before expiring. During this period, reads will ignore the tombstoned columns or rows. However, if data is reinserted while tombstones are still active, these new writes may overwrite some of the tombstoned data, leading to potential data resurrection.

#### Example Scenario

#### Cons
ider a scenario where you delete a row with a TTL set to 3 hours:

A tombstone is written immediately.
After 1 hour, a reinsertion occurs.
Since the tombstone has not yet expired (TTL = 3 hours), some columns may still be accessible during reads until the tombstone expires.

#### Common Mistakes
and Follow-Ups
Incorrect Assumptions: Assuming that deleting a row immediately removes all its data is a common mistake. Understanding that tombstones are used for marking rows as deleted but not physically removing them is crucial.
TTL Misconfiguration: Setting incorrect TTL values can lead to prolonged visibility of tombstoned data.
Performance Impact: Frequent writes and compaction operations due to many tombstones can impact performance.

#### Closing Summary

In summary, Cassandra uses logical deletes through tombstones rather than physical deletions. Tombstones are temporary markers that need to expire before their data is truly removed. A naive 'delete then reinsert' pattern can lead to data resurrection because it may not immediately clear all tombstoned columns or rows from storage.

#### Common Mistakes/Follow-Ups
Ensure you understand the TTL mechanism and its impact on data visibility.
Be cautious with reinsertion patterns that might overlap with active tombstones.
Monitor performance impacts of many tombstones and plan for timely compaction to avoid unnecessary resource consumption.

**Code:**

```python
# Example Cassandra CQL statement for deletion with a TTL
cql_query = "DELETE FROM users WHERE user_id = 123 USING TTL 10800"
```

