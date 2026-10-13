---
type: "practice"
difficulty: "Expert"
---

# Explain how partition-level (partitioned) bloom filters and per-block filters differ, and why RocksDB moved toward partitioned filters for large SSTables. What problem with monolithic filters did this solve?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Storage in LSM Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand the differences between partition-level (partitioned) bloom filters and per-block filters in the context of databases like RocksDB, we first need to grasp the core concepts of bloom filters and their role in handling large-scale data efficiently. Bloom filters are probabilistic data structures used for testing whether an element is a member of a set. In database systems, they are often employed to quickly check if a piece of data exists before performing a more costly disk I/O operation.

Partition-level (partitioned) bloom filters and per-block filters are two different strategies employed in RocksDB to manage the overhead and efficiency of bloom filters as the size of SSTables (Sorted String Table files) increases. The main difference lies in their scope and how they handle false positives at scale.

#### Triage

The question is probing our understanding of:

Bloom Filters: Their basic functionality and why they are used.
SSTable Structure and Operations: Understanding the structure of SSTables and how operations on them can benefit from bloom filters.
Partition-Level vs Per-Block Bloom Filters: The differences in their implementation, advantages, and disadvantages.
RocksDB's Motivation for Moving to Partitioned Filters: The specific problem that monolithic filters couldn't solve as the data size grew.

#### First Principles Solution
Monolithic Bloom Filters

In a traditional approach, a single bloom filter is used across an entire SSTable. This means every block in the SSTable shares the same bloom filter, which can lead to several issues:

False Positive Rate: As the size of the SSTable increases, the false positive rate (FPR) of the bloom filter also increases. This can lead to unnecessary disk I/O operations.
Update Overhead: Updating a monolithic bloom filter is costly as it requires rewriting the entire filter when new data is inserted or deleted.
Partition-Level Bloom Filters

RocksDB introduced partition-level bloom filters to address these issues:

Reduced False Positive Rate: By dividing the SSTable into smaller partitions, each with its own bloom filter, the overall false positive rate can be reduced.
Localized Updates: Updating a single partition's bloom filter is less resource-intensive compared to updating an entire SSTable.
Per-Block Bloom Filters

Per-block filters take this a step further by placing a bloom filter in each block of the SSTable:

Even Lower False Positive Rate: Each block has its own filter, which can significantly reduce false positives.
Fine-Grained Management: This approach allows for very fine-grained control over the trade-off between memory usage and accuracy.

#### Closing Summary

RocksDB moved toward partitioned filters because monolithic filters suffer from increasing false positive rates as SSTables grow in size, leading to unnecessary disk I/O operations. Partition-level and per-block bloom filters solve this problem by reducing the overall false positive rate and allowing for more efficient updates.

#### Common Mistakes/Follow-Ups
Overlooking the False Positive Rate: Sometimes engineers focus too much on memory usage without considering how it affects performance.
Ignoring Update Overhead: Monolithic filters can be updated very inefficiently, especially in large databases.
Not Understanding the Scale Impact: The problem of false positives and updates becomes more critical as the size of the database increases.

#### Example

For a 10GB SSTable with 1 million keys, if each block is 64KB, there would be approximately 157 blocks. If we use per-block filters, we can update them independently without affecting other blocks, which could lead to more efficient operations.

**Comparison:**

| Filter Type | Partitions | False Positive Rate | Update Overhead |
|-------------|------------|--------------------|-----------------|
| Monolithic  | N/A        | Increases          | High            |
| Partitioned | Yes        | Decreases          | Lower           |
| Per-Block   | Yes (per block) | Lowest   | Medium         |

**Code:**

```
CPP
Copy
// Example of a simplified partitioned bloom filter in C++
class PartitionedBloomFilter {
public:
    std::vector<BloomFilter> filters; // Vector of bloom filters per partition

    void addKey(const std::string& key) {
        int partition = hash(key); // Determine the partition based on hash
        filters[partition].addKey(key);
    }

    bool mightContainKey(const std::string& key) {
        int partition = hash(key);
        return filters[partition].mightContainKey(key);
    }
};
```

**Example input / output:**

```text
Input: A 10GB SSTable with 1 million keys and a block size of 64KB.
Output:
- Partitioned bloom filters: 157 filters (one per partition)
- Monolithic bloom filter: single filter for the entire SSTable

RocksDB: Using partitioned filters would reduce false positives and allow for efficient updates.
```

**Data:**

```json
{
  "sstables": [
    {
      "size": "10GB",
      "keyCount": "1,000,000",
      "blockSize": "64KB",
      "partitions": 157
    }
  ]
}
```

**Flow:**

1. Monolithic Bloom Filter
2. Increases FPR
3. High Update Overhead
4. Partitioned Bloom Filters
5. Divides SSTable into partitions
6. Lower FPR per partition
7. Lower Update Overhead foreach partition
8. Per-Block Bloom Filters
9. Divides SSTable into blocks
10. Least False Positive Rate
11. Medium Update Overhead perblock

