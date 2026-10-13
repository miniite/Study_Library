---
type: "practice"
difficulty: "Expert"
---

# An interviewer probes: "you picked LSM for write throughput — what breaks at 100x scale that doesn't break at 10x?" Reason through where compaction, bloom-filter memory, and open-file/SSTable counts hit non-linear cliffs, and how you'd architect around each.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-2/3 · [↑ LSM Write Path WAL and Compaction](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/LSM_Write_Path_WAL_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When considering a database system at high scale (e.g., 100x), the behavior of different storage mechanisms can significantly diverge. Level-Structured Merge (LSM) databases, like Apache Cassandra or CockroachDB, are known for their high write throughput and eventual consistency. However, as we scale up to extremely large datasets, several components—compaction, bloom filters, and open file/SSTable counts—can become bottlenecks. Let's dive into how these components behave non-linearly at 100x scale.

First, let’s break down the core idea: LSM databases store data in a log-structured manner initially and then periodically compact this data to maintain efficiency. This process involves multiple layers (memtables, SSTables) that can lead to complex interactions as the dataset grows.

#### Triage
What's Being Probed

The interviewer is interested in understanding how the non-linear behavior of these components affects performance at very large scales. Specifically:

Compaction: The frequency and efficiency of compaction.
Bloom Filters: Memory usage and false positives.
Open File/SSTable Counts: Impact on disk I/O and metadata management.

#### First-Principles Solution
Compaction

At 10x scale, compaction operates relatively smoothly because the amount of data being managed is manageable. However, at 100x scale:

Frequency of Compaction: More frequent compactions are needed to keep the system efficient.
Compaction Efficiency: As more data accumulates, the efficiency of compactions can degrade if not optimized. For example, full compactions (where all SSTables are merged) become more resource-intensive and time-consuming.

Key Insight: At 100x scale, compaction can become a significant bottleneck due to increased metadata management overhead and higher I/O demands.

Bloom Filters

Bloom filters are used for efficient data access by quickly ruling out non-existent keys. At smaller scales:

Memory Usage: Bloom filters consume less memory.
False Positives: The probability of false positives is lower.

However, at 100x scale:

Memory Blowup: As the dataset grows, the memory required for bloom filters can increase significantly, leading to higher CPU and memory overhead.
False Positive Rate: While the rate remains relatively low, the sheer number of operations means more frequent false positives, which can lead to additional I/O.

Key Insight: At 100x scale, bloom filter memory usage becomes a critical concern, potentially leading to performance degradation due to increased cache misses and CPU load.

Open File/SSTable Counts

SSTables (Sorted String Table) are the on-disk storage format in LSM databases. The number of SSTables affects disk I/O, metadata management, and overall performance:

Disk I/O: More SSTables mean more frequent I/O operations.
Metadata Management: Managing a large number of open files can be resource-intensive.

At 100x scale:

I/O Overhead: The increased number of SSTables leads to higher disk I/O, which can become a bottleneck.
Metadata Management: Metadata structures like manifest files and tombstones need to be managed more frequently, increasing the overhead.

Key Insight: At 100x scale, open file counts and SSTable management can lead to significant performance degradation due to increased metadata and I/O overhead.

Architecture Around Each Component
Compaction
Optimized Compaction Strategies: Implement adaptive compaction policies that balance frequency and efficiency.
Parallelism: Use more concurrent threads for compactions to reduce the overall impact on write throughput.
Tiered Storage: Use tiered storage solutions (e.g., SSDs for frequently accessed data) to optimize I/O.
Bloom Filters
Dynamic Tuning: Automatically adjust bloom filter sizes based on current load and memory constraints.
Error Tolerance: Accept a higher false positive rate if the trade-off in reduced memory usage is justified by overall performance gains.
Open File/SSTable Management
Efficient Metadata Handling: Optimize metadata structures to reduce overhead.
Throttling I/O Operations: Implement I/O throttling mechanisms to manage concurrent access and prevent overwhelming disk resources.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while LSM databases excel at high write throughput, the non-linear behavior of compaction, bloom filters, and open file/SSTable counts can become significant bottlenecks at 100x scale. By understanding these components, we can architect solutions that optimize performance without compromising on scalability.

#### Common Mistakes
Overlooking the impact of compaction frequency, underestimating memory requirements for bloom filters, and not properly managing open files and SSTables.

#### Follow-ups
might include:

#### Trade-offs
between consistency and availability: How does the design balance these at such scales?
Advanced Compaction Techniques: What are some advanced strategies to optimize compactions (e.g., tiered compactions)?
Alternative Storage Strategies: How do other storage mechanisms handle similar challenges?

**Comparison:**

| Component | 10x Scale | 100x Scale |
|-----------|----------|-----------|
| Compaction Frequency | Weekly | Every Hour |
| Bloom Filter Memory Usage | Low | High |
| Open File Count | Manageable | High |

**Code:**

```python
# Example of an adaptive compaction policy in pseudocode
def adaptive_compaction_policy(current_sstables, memory_usage):
    if memory_usage > threshold:
        trigger_full_compaction()
    else:
        schedule_background_compactions()
```

**Example input / output:**

```text
Input:
- Current dataset size: 1TB
- Expected scale factor: 100x

Output:
- Compaction frequency increased to every hour
- Bloom filter memory usage optimized to 5GB
- Open file count reduced by implementing tiered storage and I/O throttling
```

**Data:**

```yaml
# Sample data schema for SSTable metadata
sstable_metadata:
  - filename: "data-0123456789"
    size: 10MB
    last_modified: "2023-10-01T12:00:00Z"
```

**Flow:**

1. Yes
2. No
3. Miss
4. Hit
5. Initial Data Write
6. Memtable Full
7. Compaction Triggered?
8. Full Compaction
9. Background Compactions
10. Write Request
11. Bloom Filter Hit?
12. Bloom Filter False Positive
13. Disk I/O for SSTable Read
14. Disk I/O for SSTable Read

