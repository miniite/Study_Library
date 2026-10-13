---
type: "practice"
difficulty: "Deep"
---

# Explain how an aggregate query (COUNT/SUM/GROUP BY) is executed across shards. What partial-aggregation and final-merge steps happen, and where does a DISTINCT or ORDER BY force extra data movement you should watch for in the plan?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Distributed Joins in MPP Systems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Distributed_Joins_in_MPP_Systems/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a distributed system to handle aggregate queries (such as COUNT, SUM, or GROUP BY), it’s crucial to understand how these operations are executed across multiple shards. The process typically involves partial aggregation, merging of partial results, and sometimes additional data movement due to DISTINCT or ORDER BY clauses.

First, let's break down the core steps:

Partial Aggregation: Each shard independently executes the aggregate operation on its local data.
Merging Partial Results: The system then consolidates these local results into a final aggregated result.
Handling DISTINCT and ORDER BY: These operations can introduce additional complexity, particularly in terms of data movement.

Now, let's delve deeper into each step with an example.

#### Triaging the Question

The interviewer is probing your understanding of how distributed systems handle aggregate queries efficiently while maintaining correctness and performance. The key concepts include partial aggregation, merging processes, and the impact of DISTINCT and ORDER BY.

#### Assumptions

We have a distributed database with multiple shards.
Each shard has its own local data that is partitioned based on some criteria (e.g., hash(key)).
The system supports efficient partial aggregation.

#### First-Principles Solution
1. Partial Aggregation

Each shard performs the aggregate operation locally:

For a simple COUNT or SUM, each shard counts or sums its local data.
For a GROUP BY query, shards group their local data based on the grouping keys and compute partial aggregates.

For example, suppose we have a distributed database of user transactions, partitioned by user ID. A query to find the total number of transactions per user might look like this:

SQL
SELECT user_id, COUNT(*) AS count FROM transactions GROUP BY user_id;

Each shard would independently execute its part of the GROUP BY and COUNT operations.

2. Merging Partial Results

After local aggregation, partial results from each shard need to be merged into a global result set:

If shards are storing intermediate results (like partial counts), they can send these directly to a coordinator node or another designated aggregator.
Aggregators then combine these partial results using appropriate algorithms (e.g., summing partial sums for SUM).

Using the previous example, if each shard sends its partial count of transactions per user back to the coordinator:

SQL
Shard 1: {user_id: 101, count: 5}
Shard 2: {user_id: 101, count: 3}, {user_id: 102, count: 7}

The aggregator would then sum these values:

SQL
{user_id: 101, count: 8}, {user_id: 102, count: 7}
3. Handling DISTINCT and ORDER BY
DISTINCT: This clause requires additional data movement because it forces the system to de-duplicate values across shards. If user_id is used in a DISTINCT clause, each shard must send its unique user IDs to the coordinator for deduplication.

For example:

SQL
SELECT DISTINCT user_id FROM transactions;

Each shard would need to send its unique users back to the coordinator, which then removes duplicates and returns the final set.

ORDER BY: This can also introduce extra data movement. The system needs to combine results from all shards before applying the sort operation. Each shard sends its local sorted or partially sorted result back to a coordinator, which merges these into one large, ordered dataset.

For example:

SQL
SELECT user_id, COUNT(*) AS count FROM transactions GROUP BY user_id ORDER BY count DESC;

Shards would send their partial results like this:

SQL
Shard 1: {user_id: 101, count: 5}
Shard 2: {user_id: 101, count: 3}, {user_id: 102, count: 7}

The coordinator then merges these and applies the ORDER BY:

SQL
{user_id: 102, count: 7}, {user_id: 101, count: 8}

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, distributed aggregate queries involve local partial aggregation followed by merging of results. DISTINCT and ORDER BY can introduce significant data movement that needs to be carefully managed to avoid bottlenecks.

#### Common mistakes
include:

Failing to handle DISTINCT properly by not de-duplicating across shards.
Overlooking the need for efficient merge algorithms when combining partial results.
Ignoring the impact of ORDER BY on performance and data movement, leading to suboptimal query plans.

#### Follow-ups
might involve discussing specific optimizations or trade-offs related to these operations in a distributed setting. For instance, how does sharding key choice affect aggregation efficiency? How can caching be leveraged to improve performance?

**Comparison:**

| Shard | Partial Results |
|-------|-----------------|
| Shard 1 | user_id: 101, count: 5; user_id: 102, count: 3 |
| Shard 2 | user_id: 101, count: 3; user_id: 102, count: 7 |

**Code:**

```sql
-- Example SQL queries for aggregate functions
SELECT user_id, COUNT(*) AS count FROM transactions GROUP BY user_id;
SELECT DISTINCT user_id FROM transactions;
SELECT user_id, COUNT(*) AS count FROM transactions GROUP BY user_id ORDER BY count DESC;
```

**Example input / output:**

```text
Input:
- Shard 1: [ {user_id: 101, count: 5}, {user_id: 102, count: 3} ]
- Shard 2: [ {user_id: 101, count: 3}, {user_id: 102, count: 7} ]

Output:
- Final Aggregated Result:
  - {user_id: 101, count: 8}
  - {user_id: 102, count: 10}

Input for DISTINCT:
- Shard 1: [ {user_id: 101}, {user_id: 103} ]
- Shard 2: [ {user_id: 104}, {user_id: 105} ]

Output for DISTINCT:
- Final Result:
  - {user_id: 101}
  - {user_id: 103}
  - {user_id: 104}
  - {user_id: 105}

Input for ORDER BY:
- Shard 1: [ {user_id: 101, count: 5}, {user_id: 102, count: 3} ]
- Shard 2: [ {user_id: 101, count: 3}, {user_id: 102, count: 7} ]

Output for ORDER BY:
- Final Result:
  - {user_id: 102, count: 10}
  - {user_id: 101, count: 8}
```

**Data:**

```json
{
  "transactions": [
    {"user_id": 101, "amount": 50},
    {"user_id": 102, "amount": 30},
    {"user_id": 101, "amount": 40},
    {"user_id": 102, "amount": 70}
  ]
}
```

**Flow:**

1. Shard 1
2. Aggregate
3. Send partial result toCoordinator
4. Shard 2
5. Aggregate
6. Send partial result toCoordinator
7. Coordinator
8. Merge results
9. Return final aggregated result
10. Shard for DISTINCT
11. Collect unique user_ids
12. Send unique ids toCoordinator
13. Coordinator
14. Deduplicate and return

