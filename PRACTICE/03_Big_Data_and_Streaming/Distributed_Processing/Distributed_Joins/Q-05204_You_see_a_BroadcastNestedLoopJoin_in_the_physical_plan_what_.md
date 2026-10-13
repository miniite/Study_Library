---
type: "practice"
difficulty: "Deep"
---

# You see a BroadcastNestedLoopJoin in the physical plan — what caused it and why is it dangerous?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Verifying Join Strategy in Spark UI](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Distributed_Joins/Verifying_Join_Strategy_in_Spark_UI.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you encounter a BroadcastNestedLoopJoin in the physical execution plan of your Spark job, it indicates that one dataset (the smaller or broadcasted dataset) needs to be repeatedly scanned and joined with every row of the larger dataset. This join strategy can lead to significant performance issues because it requires many repeated operations.

To understand why this is dangerous, let's first break down the mechanics of a BroadcastNestedLoopJoin. The key factors at play are the size of the two datasets involved: one must be small enough to fit into memory and broadcasted to all nodes in the cluster, while the other dataset remains as-is. The smaller dataset (broadcasted) is then iterated over multiple times for each row in the larger dataset.

#### Triage
What's Being Probed

The interviewer is probing your understanding of Spark's execution model, join strategies, and their performance implications. They are also testing if you recognize potential bottlenecks in distributed systems and how they manifest in query execution plans.

#### Assumptions
The smaller dataset (broadcasted) fits into memory.
The larger dataset does not fit in memory or has a large number of rows.
Both datasets have non-trivial join conditions.

#### First-Principles Solution
Step 1: Analyze the Physical Plan

To diagnose and resolve issues, start by examining the detailed execution plan:

Input Dataset
Broadcasted Smaller Dataset
Larger Dataset
Nested Loop Join
Step 2: Understand the Mechanism

A BroadcastNestedLoopJoin works as follows:

Broadcast Smaller Dataset: The smaller dataset is loaded into memory on each node of the cluster and broadcasted.
Repeat Operation: For every row in the larger dataset, the join condition with the entire broadcasted dataset is checked.
Step 3: Evaluate Performance

Performance issues arise because this operation involves repeated scans of the small dataset:

Time Complexity: (O(n \times m)), where (n) is the number of rows in the smaller dataset and (m) is the number of rows in the larger dataset.
Memory Usage: The broadcasted data must fit into memory on every node, which can lead to out-of-memory errors.
Common Scenarios
Small Dataset with Large Joins: When a small dataset is joined repeatedly with a large one, leading to multiple repeated operations.
Inefficient Data Distribution: Poor distribution of the smaller dataset across nodes, causing imbalanced workloads.

#### Closing Summary
and Mistakes

A BroadcastNestedLoopJoin can be dangerous due to its high computational cost and memory overhead. It is often used as a fallback when a more efficient join strategy cannot be applied (e.g., broadcasted dataset doesn't fit into memory). To mitigate this, consider:

Broadcast Join Optimization: Use broadcastJoin or broadcastLeftOuterJoin with the appropriate configuration to ensure the smaller dataset fits.
Alternative Joins: Prefer distributed joins like hashJoin when possible.

#### Common Mistakes
Overusing Broadcast Joins: Applying them without considering the size of the datasets and potential memory constraints.
Ignoring Join Order: Not optimizing join order, leading to suboptimal plans.

**Comparison:**

```
Join Type	Time Complexity	Memory Usage
BroadcastJoin	O(n * m)	Small (fits in memory)
HashJoin	O(m + n log n)	Large
```

**Code:**

```scala
// Example in Spark Scala
val smallDF = spark.read.parquet("path/to/small/dataset")
val largeDF = spark.read.parquet("path/to/large/dataset")

// Using broadcast join with a smaller dataset that fits into memory
smallDF.broadcastJoin(largeDF, Seq("key"))
```

**Example input / output:**

```text
Input:
smallDF = small dataset with 5000 rows
largeDF = large dataset with 1,000,000 rows

Output:
BroadcastNestedLoopJoin plan executed, but not optimized.
```

**Data:**

```json
{
  "smallDataset": {
    "size": "10 MB",
    "rows": 5000,
    "columns": ["id", "value"]
  },
  "largeDataset": {
    "size": "1 GB",
    "rows": 1000000,
    "columns": ["id", "category", "value"]
  }
}
```

**Flow:**

1. Input
2. Broadcast Smaller Dataset
3. Larger Dataset
4. Nested Loop Join

