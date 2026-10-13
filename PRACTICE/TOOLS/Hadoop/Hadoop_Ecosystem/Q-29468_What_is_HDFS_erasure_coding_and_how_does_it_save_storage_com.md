---
type: "practice"
difficulty: "Deep"
---

# What is HDFS erasure coding and how does it save storage compared to three-way replication?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ HDFS Data Integrity](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/HDFS_Data_Integrity.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview
Erasure coding in HDFS (Hadoop Distributed File System) is a data protection technique that improves storage efficiency by reducing redundancy, especially in large-scale distributed systems like those used for big data processing.

#### Triage
The question asks us to explain how HDFS erasure coding works and compare it with three-way replication. This involves understanding the principles of both methods and then evaluating their effectiveness in terms of storage overhead.

#### First-Principles Solution

Three-Way Replication: In traditional Hadoop, each block is replicated three times across different nodes within the cluster to ensure data availability and fault tolerance. This approach guarantees that even if a node fails, the data remains accessible.
Erasure Coding in HDFS: Erasure coding takes a different approach by splitting a file into smaller segments (called fragments) and generating parity data. These fragments are then distributed across the nodes, with each node storing multiple fragments. The key advantage is that not all fragments need to be replicated three times—only enough to ensure data can be reconstructed if some fragments are lost.
Redundancy Reduction: By using erasure coding, HDFS can store the same amount of usable data with significantly fewer duplicate copies compared to replication. For example, a 100% redundancy requirement (three-way replication) means storing three full copies, whereas erasure coding might require only two or even one additional parity fragment per block.
Performance Impact: While erasure coding reduces storage overhead, it does introduce some computational overhead due to the process of encoding and decoding data fragments. However, in large-scale distributed systems where storage costs are high, this trade-off can result in substantial savings.

#### Closing Summary
+ Common Mistakes/Follow-Ups: Erasure coding is a powerful technique that optimizes storage efficiency by reducing redundancy compared to traditional three-way replication. It's particularly beneficial in scenarios where storage costs are critical, but it comes with an initial computational overhead and requires careful configuration to balance performance and fault tolerance.

#### Common Mistakes

Overlooking the trade-off between storage savings and computational overhead.
Assuming that erasure coding automatically provides the same level of data availability as three-way replication without considering the specific failure scenarios.
Neglecting to adjust the fragment size and parity strategy for optimal results in different use cases.

#### Follow-ups
could include discussing specific configurations, performance metrics, and real-world examples where HDFS with erasure coding has been successfully implemented.

**Comparison:**

```
Replication vs. Erasure Coding	Three-Way Replication	Erasure Coding (Example)
Redundancy	3x full copies	Smaller number of copies, additional parity fragments
Storage Overhead	Higher	Lower
Computational Overhead	Low	Higher
Fault Tolerance	High	Medium (depends on policy)
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Input File] --> B{Fragmentation};
    B --> C1[Generate Fragments];
    B --> C2[Generate Parity Data];
    C1 --> D1[Store Fragments on Nodes];
    C2 --> D2[Store Parity Data on Nodes];
    D1 --> E[Reconstruct if Needed];
    D2 --> F[Reconstruct if Needed];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Original File: "example.txt" of size 10MB |
| Erasure Coding Policy: "6+3,RS(6,9)"    | Fragments and Parity Data Distributed Across Nodes
| Storage Requirement: ~25% more than original file for parity data |

**Data:**

```yaml
# Example Configuration for Erasure Coding in HDFS
policies:
  - policy_name: Default
    erasure_code_policy:
      name: "6+3,RS(6,9)"
      type: REPLICATION
      num_data_replicas: 6
      num_parity_replicas: 3
```

**Flow:**

1. Yes
2. No
3. Read Data
4. Determine if Fragment is Local
5. Directly Read from Node
6. Fetch Fragment via Network
7. Reconstruct MissingFragments Using Parity
8. Return Reconstructed Data toClient

