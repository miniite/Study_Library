---
type: "practice"
difficulty: "Expert"
---

# How do you architect the primary-key and merge-on-read strategy for a lakehouse table fed by CDC upserts from a sharded OLTP source with 500M mutations/day? Where does merge-on-read's read amplification force you back to copy-on-write, and what's the compaction cadence?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ Real-time OLAP Serving Layer](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Real-time_OLAP_Serving_Layer.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When architecting a lakehouse table that ingests change data capture (CDC) upserts from a sharded OLTP source with 500 million mutations/day, one of the key challenges is ensuring efficient and reliable handling of primary keys while managing read amplification. The strategy of merge-on-read (MOR) can help minimize write operations but may introduce significant read amplification, potentially forcing us back to copy-on-write (COW) in certain scenarios. Compaction cadence plays a crucial role in balancing these trade-offs.

First, let's break down the key components and challenges:

Sharded OLTP Source: This implies data is spread across multiple shards or nodes.
500M Mutations/Day: High mutation rate indicates frequent updates.
CDC Upserts: Real-time updates require handling insertions and updates efficiently.
Lakehouse Table: A hybrid architecture combining the best of data lakes (unstructured, wide) and data warehouses (structured, deep).

#### Triaging the Question

The core of the question revolves around:

Efficiently managing primary keys in a sharded environment.
Ensuring that merge-on-read doesn't lead to excessive read amplification.
Determining when copy-on-write is necessary due to compaction.

#### Assumptions
and constraints include:

Data volume: High mutation rate (500M/day).

#### Cons
istency: Real-time or eventual consistency based on business requirements.
Performance: Optimizing for both write and read performance.

#### First-Principles Solution
Primary Key Strategy
Global Unique Identifier (GUID): Use a globally unique identifier as the primary key to avoid conflicts between shards.
Hashing: Hash shard IDs with GUIDs to distribute records evenly across the lakehouse table.
Time-Based Versioning: For each mutation, include a timestamp or version number to track changes.
Merge-On-Read (MOR) Strategy
Incremental Load: Only read from shards where mutations have occurred.
Materialized Views: Create materialized views for common queries to reduce read amplification.
Caching: Implement caching mechanisms to minimize repeated reads of the same data.
Read Amplification and Compaction Cadence
Read Amplification: MOR can lead to multiple reads per write, especially when dealing with wide schemas or complex queries.
Compaction: To mitigate read amplification, implement a compaction strategy that reduces the number of files and improves query performance.
Determining When to Switch to Copy-On-Write (COW)
Read Amplification Threshold: Define a threshold where frequent reads outweigh the benefits of MOR.
Query Patterns: Analyze common query patterns to identify scenarios where COW provides better performance.
Compaction Cadence: Develop a compaction cadence that balances write amplification and read performance.

#### Closing Summary

In summary, for a lakehouse table with 500M mutations/day from a sharded OLTP source, the primary-key strategy should use GUIDs and hashing to ensure global uniqueness. Merge-on-read can be efficient but must be managed to avoid excessive read amplification through incremental loads, materialized views, and caching. Compaction is crucial for maintaining performance as the data grows. In cases of high read amplification, switching to copy-on-write may be necessary, with careful consideration of compaction cadence.

#### Common Mistakes/Follow-Ups
Over-relying on MOR: Not accounting for excessive read amplification can degrade query performance.
Insufficient Compaction: Poorly defined compaction strategies can lead to storage inefficiencies and increased write amplification.
Ignoring Query Patterns: Failing to analyze common queries can result in suboptimal data organization.

**Comparison:**

 | Shard ID | Files |
 | --- | --- |
 | 1-50 | 200 |
 | Shard ID | Files |
 | --- | --- |
 | 1 | 500 |

**Code:**

```sql
-- Example of a materialized view creation for frequent queries
 CREATE MATERIALIZED VIEW sales_summary AS
 SELECT customer_id, SUM(amount) as total_sales
 FROM raw_transactions
 GROUP BY customer_id;
```

**Example input / output:**

```text
-- Example input and output for a compaction operation
 Input:
 Before compaction (5 shards, 200 files each)
 After compaction (1 shard, 500 compacted files)

 Output:
 Compacted data reduced by 75%, improving query performance.
```

**Data:**

```json
{
   "shard_config": {
     "shards": [
       {"id": 1},
       {"id": 2},
       ...
       {"id": 50}
     ]
   },
   "compaction_policy": {
     "interval_hours": 24,
     "threshold_files": 1000
   }
 }
```

**Flow:**

1. Yes
2. No
3. Shard Mutation
4. Is Compaction Needed?
5. Compaction
6. Reduce File Count, ImprovePerformance
7. Merge On Read
8. Handle High ReadAmplification

