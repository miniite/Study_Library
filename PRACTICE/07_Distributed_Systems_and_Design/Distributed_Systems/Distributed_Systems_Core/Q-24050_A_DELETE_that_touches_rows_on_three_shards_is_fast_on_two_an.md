---
type: "practice"
difficulty: "Deep"
---

# A DELETE that touches rows on three shards is fast on two and slow on one. How would you use the distributed execution plan and per-node stats to decide whether it's data skew a missing local index or a straggler shard?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Distributed Joins in MPP Systems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a DELETE operation that exhibits different performance across multiple shards, it’s crucial to diagnose whether the issue stems from data skew, the absence of an appropriate local index, or a problematic shard (straggler). By leveraging distributed execution plans and per-node statistics, we can make informed decisions. Here’s how to approach this problem:

#### Overview
We need to analyze the performance metrics to determine if one shard is significantly slower than the others due to data skew, an unindexed column, or a straggler node.

#### Triage
The core of the question revolves around identifying the root cause of uneven DELETE performance across shards. This involves understanding the execution plan and examining per-node statistics for insights.

Step-by-Step Solution:

Examine the Execution Plan: Review the distributed query execution plan to understand how the data is being partitioned and processed by each shard.
Analyze Per-Node Statistics: Look at the stats from each shard, such as the number of rows deleted, the time taken, and any errors or warnings reported.
Identify Data Skew: If one shard has significantly more or less data than others, it could indicate data skew.
Check for Missing Indexes: A local index can greatly speed up DELETE operations by allowing faster row lookups. Ensure that the columns involved in the DELETE are indexed on all shards.
Detect Straggler Shards: If one shard is significantly slower without a clear reason (data skew or missing index), it might be a straggler.

#### Closing Summary
and Common Mistakes:

The key to solving this problem lies in combining the execution plan with per-node statistics.

#### Common mistakes
include not thoroughly examining all possible causes, such as failing to consider data skew or missing indexes.
Always start by checking for obvious issues before diving into more complex diagnostics.

**Comparison:**

| Shard | Rows Deleted | Time Taken (ms) | Index Usage |
|-------|-------------|----------------|------------|
| 1     | 200         | 100            | Yes        |
| 2     | 300         | 90             | Yes        |
| 3     | 500         | 1800           | No         |

**Code:**

```sql
-- Example SQL for a distributed DELETE operation
DELETE FROM orders WHERE order_id IN (SELECT id FROM orders WHERE customer_id = 12345 AND order_date > '2023-01-01');
```

**Example input / output:**

```

Given the above data, we can see that shard_3 is significantly slower. The number of rows deleted (500) and the time taken (1800ms) are much higher than in other shards.

Shard 1
Rows: 200, Time: 100ms
Shard 2
Rows: 300, Time: 90ms
Shard 3
Rows: 500, Time: 1800ms, NoIndex
```

**Data:**

```json
{
  "shard_1": {
    "rows_deleted": 200,
    "time_taken_ms": 100,
    "index_usage": true
  },
  "shard_2": {
    "rows_deleted": 300,
    "time_taken_ms": 90,
    "index_usage": true
  },
  "shard_3": {
    "rows_deleted": 500,
    "time_taken_ms": 1800,
    "index_usage": false
  }
}
```

**Flow:**

1. Yes
2. No
3. No
4. Yes
5. Start
6. Is the data equally distributed?
7. Are there missing indexes onall shards?
8. straggler shard
9. Data Skew
10. Check for straggler node

