---
type: "practice"
difficulty: "Intermediate"
---

# Your team is choosing between InnoDB and RocksDB (MyRocks) for a new metadata store. The workload is 80% reads with a hot small key range and strong latency SLOs. Argue which engine you'd pick and what would flip your decision.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree Concurrency and Latches](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_Concurrency_and_Latches.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When choosing between InnoDB and RocksDB (MyRocks) for a new metadata store with 80% read operations on a hot small key range, and stringent latency SLOs, you need to consider several factors that align closely with the workload characteristics. InnoDB is an established choice that excels in transactional consistency and durability due to its multi-version concurrency control (MVCC) and support for ACID transactions. RocksDB, on the other hand, is a high-performance key-value store designed for scenarios where write amplification can be minimized.

In this scenario, the key decision factors include:

Read Performance: Both engines are optimized for read-heavy workloads, but RocksDB often outperforms InnoDB in such cases due to its design as an append-only storage layer.
Latency SLOs: Given the emphasis on low-latency reads, a robust and predictable latency is crucial. InnoDB’s transactional overhead might introduce additional latency variations.
Hot Key Range Management: Managing hot key ranges efficiently requires minimizing cache pressure and ensuring data locality. RocksDB's compact format and bloom filters can help in this regard.

#### Triage
What the Question is Really Asking

The interviewer is probing your understanding of database internals, specifically B-Tree vs LSM-Tree architectures, and how they influence performance under different workloads. The assumptions include:

A read-heavy workload with a hot key range.
Strong latency SLOs that must be met reliably.
Step-by-Step Solution From First Principles
Read Performance: RocksDB's LSM (Log-Structured Merge) architecture is designed for high throughput and minimal write amplification, making it highly efficient in scenarios where reads dominate the workload. In contrast, InnoDB’s B-tree structure is more suited to transactional workloads with mixed read/write patterns.
Latency SLOs: For strong latency requirements, RocksDB’s ability to handle hot key ranges efficiently can lead to lower tail latencies due to its compact storage and caching mechanisms. InnoDB’s multi-versioned approach might introduce additional delays during MVCC operations.
Hot Key Range Management: Managing hot data effectively is crucial. RocksDB's bloom filters and other optimizations make it more efficient in serving hot key ranges, whereas InnoDB's B-tree structure may require more frequent disk I/Os due to its balanced tree nature.

#### Closing Summary
+ Common Mistakes/Follow-Ups

#### Closing Summary
Given the read-heavy workload with a hot small key range and strong latency SLOs, RocksDB (MyRocks) would be the preferred choice. Its architecture is optimized for high read throughput and efficient handling of hot data ranges, making it better suited to meet stringent latency requirements.

#### Common Mistakes/Follow-Ups

Forgetting about transactional guarantees: InnoDB’s ACID transactions might introduce unnecessary overhead in a purely read-heavy workload.
Underestimating RocksDB's flexibility: While RocksDB is highly performant for reads, it can also support additional features like range scans and point queries efficiently.

If the workload had more write operations or less stringent latency requirements, the decision might favor InnoDB due to its robust transactional semantics. However, with the current profile, RocksDB’s design aligns better with the needs of this particular metadata store.

**Comparison:**

```
Engine	Strengths	Weaknesses
InnoDB	ACID transactions, MVCC	Higher write amplification
RocksDB	High read throughput, Efficient hot key range management	Limited transactional guarantees
```

**Code:**

```
MERMAID
Copy
graph LR;
    A[Read-Heavy Workload] --> B[RocksDB LSM];
    A --> C[InnoDB B-tree];
    B --> D[Low Tail Latency];
    C --> E[Higher Transactional Overhead];
    D --> F[Hot Key Range Efficiency];
    E --> G[ACID Transactions];
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Read-Heavy Workload, Hot Key Range, Low SLO Latency | RocksDB (MyRocks) |

**Data:**

```json
{
  "workload": {
    "read_percentage": 80,
    "hot_key_range": true,
    "slo_latency": "Low"
  },
  "engine_choices": [
    {
      "name": "InnoDB",
      "strengths": ["ACID transactions", "MVCC"],
      "weaknesses": ["Higher write amplification", "Balanced B-tree structure"]
    },
    {
      "name": "RocksDB (MyRocks)",
      "strengths": ["High read throughput", "Efficient hot key range management"],
      "weaknesses": ["Limited transactional guarantees"]
    }
  ]
}
```

**Flow:**

1. Read-Heavy Workload
2. RocksDB LSM
3. InnoDB B-tree
4. Low Tail Latency
5. Higher Transactional Overhead
6. Hot Key Range Efficiency
7. ACID Transactions

