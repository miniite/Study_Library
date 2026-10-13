---
type: "practice"
difficulty: "Expert"
---

# What is a shard key and what happens if you pick a low-cardinality or monotonically increasing one?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1 · [↑ Composite vs Surrogate Keys](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

A shard key in distributed databases helps partition data across multiple servers to improve performance and scalability. However, selecting an inappropriate shard key—such as a low-cardinality or monotonically increasing one—can significantly degrade query performance.

#### Triage

The question is probing your understanding of how shard keys work and the potential pitfalls of using them improperly. Specifically, it tests whether you can explain what happens when choosing a shard key with low cardinality (i.e., few distinct values) or that increases monotonically over time. These choices can lead to suboptimal performance and increased load on some shards.

#### First-Principles Solution
What is a Shard Key?

A shard key is used in distributed databases to partition data into smaller, more manageable chunks called shards. The shard key is a column or combination of columns that determines which shard a particular record belongs to. Typically, the goal is to distribute records evenly across shards to ensure balanced load and efficient query execution.

Low-Cardinality Shard Key

A low-cardinality shard key has few distinct values. For example, if you choose an attribute like customer_id as the shard key in a customer database, there will be only one or very few shards, leading to poor partitioning. Here’s why this is problematic:

Uneven Data Distribution: With a low-cardinality key, records with the same shard key value (e.g., customer_id) end up on the same shard, making it impossible for queries spanning multiple customers to be distributed evenly.
Hot Spotting: Some shards will receive disproportionately more traffic than others, leading to bottlenecks and performance degradation.
Monotonically Increasing Shard Key

A monotonically increasing shard key is one that strictly increases in value over time (e.g., timestamp, ID). While this might seem beneficial because records are spread out across shards as new data arrives, it can also lead to problems:

Hot Spotting: As the shard key continues to increase, a single or a small set of shards will receive most of the write operations. This uneven distribution can cause hot spots and performance issues.
Fragmentation: Over time, the database may become fragmented as older data is spread out across many shards while new data accumulates in a few.

#### Closing Summary

Choosing an inappropriate shard key, such as one with low cardinality or that increases monotonically, can lead to hot spots and uneven load distribution, which can significantly degrade performance. It’s crucial to carefully select a shard key based on the distribution of query patterns and business requirements to ensure optimal performance and scalability.

#### Common Mistakes/Follow-Ups
Ignoring Query Patterns: Not considering how queries will be executed and distributed across shards.
Not Accounting for Data Growth: Assuming that monotonically increasing keys won't lead to hot spots over time.
Overlooking Cardinality: Ignoring the number of unique values a shard key can take, leading to poor data distribution.

