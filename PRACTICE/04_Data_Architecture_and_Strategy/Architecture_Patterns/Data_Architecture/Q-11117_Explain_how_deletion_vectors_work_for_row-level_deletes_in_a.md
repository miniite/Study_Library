---
type: "practice"
difficulty: "Deep"
---

# Explain how deletion vectors work for row-level deletes in a lakehouse and the read-time cost of letting them accumulate. What compaction cadence keeps queries fast?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Vector Data in the Lakehouse](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Vector_Data_in_the_Lakehouse.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Deletion vectors are a powerful mechanism used in data architecture, particularly in lakehouses, to efficiently manage row-level deletes without physically removing the rows from storage. By marking rows as deleted rather than deleting them outright, deletion vectors offer both flexibility and cost efficiency. However, allowing these vectors to accumulate over time can introduce read-time overheads that impact query performance. The key challenge is finding an appropriate compaction cadence that balances these trade-offs.

#### Triage
Question Breakdown

The question delves into the internal workings of deletion vectors in a lakehouse environment and explores how they affect query performance. Specifically, it aims to understand:

How deletion vectors are implemented and their benefits.
The downsides of allowing deletion vectors to accumulate over time.
How to optimize read-time performance through compaction.

#### Assumptions
Lakehouse Environment: We're discussing a modern data architecture that combines data lakes with the benefits of traditional data warehouses, such as structured query languages (SQL) and optimized queries.
Deletion Vectors: These are metadata structures that track which rows have been deleted from a table.
Compaction Cadence: This refers to how frequently we clean up or compact deletion vectors to maintain optimal read-time performance.
Step-by-Step Solution
1. Understanding Deletion Vectors

In a lakehouse, deletion vectors are used to mark rows that should be ignored by queries but aren't physically removed from storage. Each row is assigned a unique version number, and when a delete operation occurs, the corresponding version numbers in the vector are updated to indicate deletion.

For example:

PLAINTEXT
Row 1: Version = 20
Row 2: Version = 20
Row 3: Version = 25 (Deleted)
Row 4: Version = 26

Deletion Vector: [25]

When a query runs, it checks the deletion vector to skip over any marked rows.

2. Read-Time Cost of Accumulating Deletion Vectors

As deletion vectors accumulate, they can introduce significant overhead during read-time operations. Each row needs to be checked against the deletion vector before processing, leading to potentially slower query performance as more deletions occur over time.

To understand this better, consider a simple example where we have 100 million rows in a table and only 2% are marked for deletion:

PLAINTEXT
Rows: 100,000,000
Deleted Rows (2%): 2,000,000

Deletion Vector Size: ~2 MB (assuming 8 bytes per version number)

Every time a query runs, it must check these 2 million rows against the deletion vector, which can be computationally expensive.

3. Compaction Cadence

Compacting the deletion vectors involves reorganizing them to reduce the read-time overhead while maintaining accuracy. The goal is to balance the trade-off between storage costs and performance.

Compaction Techniques:
Batch Compaction: Periodically merge multiple small compactions into a single larger operation, reducing the overhead of frequent compactions.
Incremental Compaction: Gradually clean up the deletion vectors over time without disrupting ongoing operations.
Hybrid Compaction: A combination of batch and incremental approaches to optimize performance and storage.
Optimization Example:

Let's say we have a table with 100 million rows, and 5% are marked for deletion. We want to compact the deletion vector every week while keeping it small enough to maintain acceptable query performance.

PLAINTEXT
Compaction Cadence: Weekly
Delete Vector Size Target: ~3 MB

Weekly Compaction Process:
1. Identify rows with recent deletions.
2. Update the delete vector.
3. Clean up old entries.
4. Store compacted version in a new segment file.
5. Replace old segments with compacted ones.

Query Optimization during Compaction:
- Use parallel processing to speed up compaction tasks.
- Apply optimizations like row-level pruning to minimize impact on ongoing queries.

#### Closing Summary

Compacting deletion vectors is crucial for maintaining efficient read-time performance in a lakehouse environment. By carefully managing the compaction cadence, we can strike a balance between storage overhead and query efficiency. The key is to perform compactions frequently enough to keep overhead low but not so often as to disrupt ongoing operations.

#### Common Mistakes/Follow-Ups
Ignoring Compaction: Not performing regular compactions can lead to significant performance degradation.
Over-compacting: Too frequent or overly aggressive compactions can introduce unnecessary write amplification and resource contention.
Inadequate Monitoring: Failing to monitor query performance and deletion vector size can result in suboptimal configurations.

For further discussion, we could explore advanced techniques like online compaction, data skew management, and integrating machine learning for dynamic compaction thresholds.

