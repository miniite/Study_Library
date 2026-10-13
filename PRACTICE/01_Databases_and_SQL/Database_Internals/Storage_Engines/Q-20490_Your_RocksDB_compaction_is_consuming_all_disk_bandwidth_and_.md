---
type: "practice"
difficulty: "Expert"
---

# Your RocksDB compaction is consuming all disk bandwidth and starving foreground reads/writes. How do you rate-limit compaction, and what's the trade-off between throttling compaction and letting read amplification grow?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ LSM Write Path WAL and Compaction](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of RocksDB compaction consuming all disk bandwidth and starving foreground reads/writes, we need to implement rate limiting on compactions while considering the trade-offs involved in doing so. This involves understanding how compaction works in RocksDB and evaluating different strategies to balance read/write performance with compaction efficiency.

TL;DR

Rate-limiting compactions is essential to prevent disk bandwidth saturation but comes at a cost of increased read amplification. By carefully tuning our compaction strategy, we can optimize the system's overall performance while minimizing disruptions to foreground operations.

#### Triage
and Assumptions
Understanding RocksDB Compaction: RocksDB uses multiple levels of Sstables and employs various compaction algorithms (e.g., LevelCompaction) to manage disk space efficiently.
Disk Bandwidth: The system has limited disk bandwidth, which is being heavily utilized by compactions.
Read/Write Requirements: There are significant foreground read/write operations that need to be prioritized.

#### First-Principles Solution
1. Compaction Types and Prioritization

In RocksDB, compactions can be categorized into two main types:

Minor Compaction: Involves merging small Sstables within the same level.
Major Compaction: Merges multiple levels of Sstables.

Priority Considerations:

Foreground Read/Write Operations: Prioritize reads over writes, as reads are generally more frequent and less sensitive to latency compared to write operations that can be batched or delayed.
Compaction Strategy Tuning: Adjust the compaction strategy (e.g., LevelCompaction settings) to give higher priority to minor compactions while deferring major compactions.
2. Rate Limiting Compactions

To rate-limit compactions, we can use a combination of RocksDB's built-in features and custom logic:

Compaction Priority: Configure the compaction scheduler to prioritize foreground operations over background compactions.
Compaction Filter: Use compaction_filter options to limit the number of Sstables involved in major compactions.
Manual Compaction Control: Implement a mechanism to manually trigger compactions based on disk usage metrics.
3. Trade-offs and Performance Metrics

The key trade-off is between:

Read Amplification: Increased read operations due to more frequent minor compactions.
Write Performance: Reduced write throughput during compaction periods, but improved overall durability and data consistency.
Detailed Steps

Adjust Compaction Priority:

PLAINTEXT
# Increase the priority of foreground operations over background compactions
[rocksdb]
  rocksdb_compaction_priority = "0"

Compaction Filter Configuration:

PLAINTEXT
[rocksdb]
  rocksdb_major_compaction_filter = "LevelCompactionFilter"
  # Set a threshold for the number of Sstables to be involved in major compactions
  levelcompaction_filter_threshold = 50

Manual Compaction Control:

PYTHON
def trigger_manual_compaction(database_path):
    with open(os.path.join(database_path, "compaction.trigger"), 'w') as f:
        f.write("major")

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary

By carefully tuning compaction settings and implementing rate limiting mechanisms, we can ensure that foreground read/write operations are not starved while still maintaining efficient disk usage. The key is to balance the compaction load with real-time operational needs.

#### Common Mistakes
Over-compacting: Frequent minor compactions can lead to higher read amplification, reducing overall performance.
Under-compacting: Insufficient compactions may cause excessive data fragmentation and write amplification.

#### Follow-Ups
Metrics Monitoring: Continuously monitor disk usage, latency metrics, and compaction rates to adjust configurations as needed.
Performance Testing: Use synthetic workloads to test the impact of different compaction strategies on real-world performance.
User Feedback: Gather feedback from application users to identify specific pain points related to compaction.

