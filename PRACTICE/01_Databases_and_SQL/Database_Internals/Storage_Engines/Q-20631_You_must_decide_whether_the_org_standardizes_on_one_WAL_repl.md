---
type: "practice"
difficulty: "Expert"
---

# You must decide whether the org standardizes on one WAL/replication paradigm (physical page-level vs logical row-level) across all engines to simplify DR and CDC. Reason through the trade-offs when B-Tree and LSM engines coexist, and what CDC pipeline you'd build on top.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ B-tree vs LSM Storage Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_vs_LSM_Storage_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In deciding whether to standardize on a single Write-Ahead Log (WAL) or replication paradigm across B-Tree and LSM engines, we must weigh several factors including performance, consistency, data durability, and operational complexity. For B-Tree and LSM coexistence, a logical row-level WAL would be more suitable, but this comes with trade-offs in terms of write amplification and performance overhead.

#### Triaging the Question

The core issue is to determine whether standardizing on one replication paradigm—either physical page-level or logical row-level—across different database engines (B-Tree and LSM) will simplify disaster recovery (DR) and change data capture (CDC). We need to understand the implications of each approach, especially considering the fundamental differences in how B-Tree and LSM engines operate.

#### Assumptions

#### Cons
istency Requirements: High consistency is critical for certain use cases.
Latency Sensitivity: Some applications require low write latency.
Storage Constraints: Limited storage resources might necessitate different replication strategies.
Operational Complexity: The goal of standardization is to reduce operational overhead and streamline management.

#### First-Principles Solution
B-Tree Engines

B-Trees are optimized for random access, making them suitable for OLTP workloads where read performance is critical. They use physical page-level writes because this aligns with their internal structure, ensuring fast access times and minimizing the number of I/O operations.

Key Considerations:

Physical Page-Level Writes: Ensure that changes are written to a fixed location in memory or storage, making recovery straightforward.
ACID Compliance: B-Trees provide strong transactional guarantees, which is essential for many enterprise applications.
Simplicity in Recovery: Physical page-level writes simplify the recovery process by ensuring that every change is atomic and recoverable.
LSM Engines

LSM (Log-Structured Merge) engines are designed for high write throughput at the expense of occasional slower read performance. They use logical row-level writes, storing changes in a log before merging them with existing data.

Key Considerations:

Logical Row-Level Writes: Changes are logged and eventually merged, which can lead to more frequent I/O operations.
Write Amplification: Frequent write amplification due to compaction processes can degrade performance over time.
CDC Support: Logical row-level writes make CDC easier to implement because changes can be captured as they happen.
Standardization Decision

Standardizing on a single WAL/replication paradigm across B-Tree and LSM engines would simplify DR and CDC, but it comes with significant trade-offs:

Performance Overhead: Physical page-level writes in an LSM environment could introduce unnecessary I/O overhead.
Write Amplification: Logical row-level writes in a B-Tree engine might lead to increased write amplification during recovery.
Operational Complexity: A single paradigm might complicate the deployment and management of different engines, which have distinct operational requirements.

Recommended Approach: For B-Tree and LSM coexistence, it is more pragmatic to maintain separate replication paradigms:

Use physical page-level WALs for B-Tree engines.
Use logical row-level WALs for LSM engines, leveraging the natural CDC support provided by these engines.
CDC Pipeline Design

Given the differences in how B-Trees and LSMs operate, a hybrid approach to CDC is recommended:

B-Tree Engines: Capture transactions directly from the transaction logs or journal files.
LSM Engines: Leverage the existing change data capture mechanisms embedded within the engines.

This hybrid approach ensures that each engine uses its strengths while maintaining consistency and performance:

For B-Trees, use a traditional CDC pipeline that captures transactions at the page level.
For LSMs, integrate with their built-in CDC mechanisms to capture row-level changes efficiently.

#### Closing Summary

In conclusion, standardizing on a single WAL/replication paradigm across B-Tree and LSM engines would introduce unnecessary complexity and potential performance bottlenecks. Maintaining separate replication paradigms—physical page-level for B-Trees and logical row-level for LSMs—aligns better with the operational characteristics of these engines and provides a more robust CDC pipeline.

#### Common Mistakes/Follow-Ups
Ignoring Engine-Specific Optimizations: Forgetting to leverage the inherent strengths of each engine can lead to suboptimal performance.
Over-Simplification: Overly simplifying the design by ignoring the nuances between B-Trees and LSMs could result in missed optimizations.
Inadequate Recovery Mechanisms: Failing to account for recovery mechanisms specific to each engine might introduce risks during DR scenarios.

**Comparison:**

```
Engine Type	Replication Paradigm	Key Considerations
B-Tree	Physical Page-Level	ACID compliance, recovery simplicity
LSM	Logical Row-Level	Write amplification, natural CDC
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Decision Point] --> B{B-Tree Engines};
    B --> C[Physical Page-Level WAL];
    A --> D{LSM Engines};
    D --> E[Logical Row-Level WAL];
```

