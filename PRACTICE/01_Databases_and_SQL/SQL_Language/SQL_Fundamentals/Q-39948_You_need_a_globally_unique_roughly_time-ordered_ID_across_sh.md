---
type: "practice"
difficulty: "Expert"
---

# You need a globally unique, roughly time-ordered ID across sharded tables while a shared sequence contends. How would you move to a Snowflake-style ID or per-shard offset sequences safely?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Idempotency Key Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of generating globally unique IDs in a distributed database environment, leveraging Snowflake's pattern is an effective approach. This method ensures that each shard has its own sequence while maintaining a roughly time-ordered ID across all shards. The core idea is to use per-shard sequences with a minimal central coordination mechanism.

The key question this problem aims to probe includes:

Understanding of distributed systems and the challenges they pose.
Knowledge of SQL for managing sequences and transactions.
Ability to handle race conditions and concurrency issues.
Understanding of trade-offs between simplicity, performance, and fault tolerance.

To solve this, we need to implement a system where each shard maintains its own sequence (offset) but periodically synchronizes with a central coordinator. This ensures that the IDs remain roughly time-ordered while minimizing contention on the shared sequence.

Here’s how you can achieve it:

Step-by-Step Solution
Shard-Level Sequences: Each shard maintains its own sequence generator to assign IDs locally.
Central Coordinator: A central service or database keeps track of the maximum ID assigned across all shards and periodically updates each shard with this information.
Synchronization Mechanism: Shards frequently sync their sequences with the central coordinator, ensuring that they do not generate overlapping IDs.

#### Implementation
Details

Shard Sequence Initialization:

Each shard initializes its sequence to a starting value. For example, Shard A might start at 10000, Shard B at 20000.

Central Coordinator Table Schema:

SQL
CREATE TABLE ShardSync (
    ShardId INT PRIMARY KEY,
    MaxSeqVal BIGINT DEFAULT 0
);

Shard Sequence Allocation and Synchronization:

When a shard needs to generate an ID, it queries the central coordinator for the latest max sequence value.
The shard then generates its next ID by incrementing its local sequence until it is greater than the last seen maximum.
Periodically, the shard sends its current max sequence back to the central coordinator and updates its sequence.

#### Example
Workflow

Shard A Requests Sequence:

SQL
-- Shard A queries for the latest max sequence value from the central coordinator
SELECT MaxSeqVal INTO @max_seq FROM ShardSync WHERE ShardId = 1;

-- Shard A starts generating IDs by incrementing its local sequence
WHILE @local_seq <= @max_seq DO
    SET @local_seq := @local_seq + 1;  -- Increment the shard's local sequence
END WHILE;

-- After a period, Shard A sends its latest max sequence to the central coordinator
UPDATE ShardSync SET MaxSeqVal = @local_seq WHERE ShardId = 1;

Central Coordinator Updates:

The central coordinator updates its table with the new maximum sequence value for each shard.
SQL
-- Central coordinator receives updates from shards and updates max sequence values
UPDATE ShardSync SET MaxSeqVal = @new_max_seq WHERE ShardId = 1;

#### Trade-offs
Simplicity vs. Performance: While this approach is simple, the periodic synchronization can introduce some latency.
Fault Tolerance: If the central coordinator fails, shards will still generate IDs locally but may experience occasional overlaps until recovery.

#### Common Mistakes
and Follow-Ups
Failing to Handle Overlaps: Ensure you have a strategy for handling potential overlaps in case of failures or miscommunications between shards.
Not Considering Edge Cases: What happens if the central coordinator is down? How do you handle shard restarts?
Scaling Issues: As the number of shards increases, the frequency and volume of synchronization requests can become a bottleneck.

**Comparison:**

```
ShardId	MaxSeqVal
1	20000
2	30000
```

**Code:**

```sql
-- Shard A initialization and sequence generation
SET @local_seq = 10000; -- Starting local sequence value

WHILE @local_seq <= (SELECT MaxSeqVal FROM ShardSync WHERE ShardId = 1) DO
    SET @local_seq := @local_seq + 1;
END WHILE;

-- Periodic update to the central coordinator
UPDATE ShardSync SET MaxSeqVal = @local_seq WHERE ShardId = 1;
```

**Example input / output:**

```
Input: Shard A requests its next sequence value.
Output: Shard A receives the latest max sequence from the central coordinator and generates its ID.
SQL
Copy
-- Shard A query for next sequence
SELECT MaxSeqVal INTO @max_seq FROM ShardSync WHERE ShardId = 1;

-- Shard A starts generating IDs
WHILE @local_seq <= @max_seq DO
    SET @local_seq := @local_seq + 1;
END WHILE;

-- Update the central coordinator with new max sequence
UPDATE ShardSync SET MaxSeqVal = @local_seq WHERE ShardId = 1;
```

**Data:**

```sql
CREATE TABLE ShardSync (
    ShardId INT PRIMARY KEY,
    MaxSeqVal BIGINT DEFAULT 0
);
INSERT INTO ShardSync (ShardId) VALUES (1), (2); -- Example shards
```

**Flow:**

1. Yes
2. No
3. Shard A
4. Queries Coordinator
5. MaxSeqVal
6. Generates IDs
7. Periodic Update
8. Send to Coordinator

