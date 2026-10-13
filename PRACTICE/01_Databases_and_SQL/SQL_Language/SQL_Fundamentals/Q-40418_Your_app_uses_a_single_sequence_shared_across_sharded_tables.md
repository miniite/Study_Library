---
type: "practice"
difficulty: "Expert"
---

# Your app uses a single sequence shared across sharded tables and inserts are contending on it. How would you replace it with per-shard offset sequences or a distributed ID scheme while keeping IDs globally unique and roughly time-ordered?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotency Key Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of contention on a shared sequence across sharded tables, we need to transition to a per-shard offset sequence or a distributed ID generation strategy that maintains global uniqueness and roughly time-ordering. The key challenge is ensuring that each shard can generate its own unique IDs without conflicting with other shards.

#### Triaging the Question

The question is probing how to handle auto-incrementing primary keys in a sharded database architecture, where contention arises due to shared sequences. We need to ensure that:

Global Uniqueness: Each generated ID must be globally unique across all shards.
Time-Ordering: The IDs should roughly maintain their order based on when they were inserted.
Scalability and Performance: The solution should not introduce significant overhead or contention issues.

#### Assumptions
We are using a database system that supports distributed transactions (e.g., PostgreSQL with its UUID sequence).
Each shard is an independent entity, capable of generating its own unique IDs.
A global clock skew can be tolerated to some extent; the solution should handle minor timing differences.

#### First-Principles Solution

To achieve our goals, we will use a distributed ID generation strategy based on offset sequences or UUIDs with incremental offsets. This approach ensures that each shard generates its own sequence of IDs, reducing contention and improving performance.

Step 1: Offsets and Timestamp-based Approach

We can maintain an "offset" table in the database where each row represents a shard and its current ID generation point. The offset table keeps track of the last ID generated for each shard.

SQL
CREATE TABLE offsets (
    shard_id INT PRIMARY KEY,
    next_id BIGINT
);
Step 2: Insertion with Offset

During an insert operation, we query the offsets table to get the current ID for the relevant shard. We then increment this ID and update the offset in the table.

SQL
BEGIN TRANSACTION;

-- Get the latest ID for the shard
SELECT next_id INTO @last_id FROM offsets WHERE shard_id = ? FOR UPDATE;

-- Generate new ID
SET @new_id = @last_id + 1;

-- Update the offset to reflect the new ID
UPDATE offsets SET next_id = next_id + 1 WHERE shard_id = ?;

INSERT INTO main_table (id, data) VALUES (?, ?);

COMMIT;
Step 3: UUID-based Approach with Incremental Offset

Alternatively, we can use a combination of UUID and an offset. Each shard generates its own unique UUID, but increments this by a small integer to ensure the ID remains roughly time-ordered.

SQL
-- Generate a UUID and increment it slightly for each shard
INSERT INTO main_table (id, data) VALUES (CONCAT(UUID(), '_', shard_id), ?);

This approach ensures that while the IDs are globally unique due to the UUID component, the incremental part helps maintain some degree of time-ordering.

#### Closing Summary

By using offset sequences or a combination of UUID and incremental offsets, we can effectively replace a shared sequence with per-shard generation strategies. This approach maintains global uniqueness and roughly preserves insertion order while significantly reducing contention and improving scalability.

#### Common Mistakes
and Follow-Ups
Forgetting Transaction Management: Ensure transactions are properly managed to avoid race conditions.
Ignoring Clock Skew: Be aware that minor clock differences between shards can lead to non-sequential ID generation. Implementing a global timestamp or using a more sophisticated synchronization mechanism might be necessary.
Scalability Considerations: Evaluate the number of shards and ensure the overhead introduced by maintaining offsets is acceptable.

**Comparison:**

| shard_id | next_id |
|----------|---------|
| 1        | 100     |

**Code:**

```sql
-- Offsets Table Creation
CREATE TABLE offsets (
    shard_id INT PRIMARY KEY,
    next_id BIGINT
);

-- Insertion with Offset Strategy
BEGIN TRANSACTION;

SELECT next_id INTO @last_id FROM offsets WHERE shard_id = ? FOR UPDATE;

SET @new_id = @last_id + 1;

UPDATE offsets SET next_id = next_id + 1 WHERE shard_id = ?;

INSERT INTO main_table (id, data) VALUES (?, ?);

COMMIT;
```

