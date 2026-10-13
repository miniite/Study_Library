---
type: "practice"
difficulty: "Deep"
---

# Explain how Spanner's split points and load-based splitting work, why monotonically increasing keys cause hotspots, and the bit-reversed/UUID mitigations you'd recommend for a high-ingest table.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Distributed Query Execution](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/NewSQL/Distributed_Query_Execution.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To explain Spanner’s split points and load-based splitting, we must first understand how Spanner handles data distribution and partitioning to ensure scalability and availability. Spanner uses a distributed key range sharding mechanism based on split points, which are chosen monotonically increasing keys in the primary index. Load-based splitting ensures that data is evenly spread across nodes by dynamically adjusting these split points.

#### Triaging the Question

The question delves into how Spanner manages data partitioning and load balancing, specifically focusing on two key aspects:

Split Points: The mechanism by which Spanner divides its data into ranges.
Load-Based Splitting: A dynamic technique used to ensure that no single node becomes a bottleneck.

Additionally, we are asked about the implications of using monotonically increasing keys and why they can cause hotspots, as well as mitigations for such issues in high-ingest tables.

#### First-Principles Solution

Spanner uses split points to partition its data into ranges. These split points are chosen from primary index values that form a unique key range. The first split point is the minimum value of the index, and subsequent split points are derived from the primary keys. Load-based splitting is an automated process where Spanner adjusts these split points over time based on the load distribution across nodes.

Monotonically Increasing Keys

Monotonically increasing primary keys ensure that data is inserted in a sequential manner. This seems ideal for partitioning, as it allows for straightforward range-based splits. However, this approach can lead to hotspots because:

Insertion Bottlenecks: As more rows are inserted into the same split point, one node may become overloaded.
Node Imbalance: If nodes handle data unevenly due to skewed primary key insertions, it can result in imbalanced load distribution.
Mitigations

To mitigate these issues, Spanner employs a few strategies:

Bit-Reversed Keys (UUID-based): UUIDs provide a more uniform distribution of keys across the range.
Hashed Partitioning: Using hash functions to distribute data evenly can further reduce hotspots and improve load balancing.

#### Closing Summary

In summary, while monotonically increasing primary keys are convenient for sequential insertion, they can cause hotspots in high-ingest scenarios. Spanner mitigates this issue by using bit-reversed UUIDs or hashed partitioning techniques. These strategies ensure a more balanced data distribution and prevent single-node bottlenecks.

#### Common Mistakes/Follow-Ups
Misunderstanding of Split Points: Often, developers confuse split points with primary keys, leading to incorrect assumptions about how data is partitioned.
Ignoring Load Distribution: Developers might not fully appreciate the importance of dynamic load-based splitting and its impact on system performance.
Overlooking Data Insertion Patterns: Not considering the specific patterns in which data is inserted can lead to suboptimal use of Spanner's features.

For further exploration, we could discuss how Spanner handles read-heavy vs write-heavy workloads or explore advanced concepts like global transactions and cross-region replication.

**Comparison:**

```
Split Point	Key Range
1	00000000-0000-0000-0000-000000000000 - 00000000-0000-0000-0000-0000000000FF
2	00000000-0000-0000-0000-000000000100 - 00000000-0000-0000-0000-0000000001FF
...	...
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Spanner] --> B1{Partitioning};
    B1 --> C[Split Points];
    B1 --> D[Load-Based Splitting];
    C --> E1(Sequential Primary Keys);
    C --> E2(Bit-Reversed UUIDs);
    C --> E3(Hashed Partitioning);
```

**Example input / output:**

```text
Input:
- Insertion of data with monotonically increasing UUIDs

Output:
- Data is distributed across split points, but nodes handling large ranges may become hotspots.
```

**Data:**

```json
{
  "splitPoints": [
    { "key": "00000000-0000-0000-0000-000000000001", "loadBased": true },
    { "key": "00000000-0000-0000-0000-000000000002", "loadBased": false }
  ]
}
```

**Flow:**

1. Data Insertion
2. Check Key
3. Bit-Reversed UUIDs
4. Hashed Partitioning
5. Load-Based Splitting
6. Evenly Distributed Data

