---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design a durable append-only event log storage engine from scratch. Would you use an LSM, a B-Tree, or a simpler segment-file design like Kafka? Justify with write pattern, read pattern, and retention.

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ B-tree vs LSM Storage Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/B-tree_vs_LSM_Storage_Engines.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a durable append-only event log storage engine from scratch, we need to consider several factors such as the write pattern (how events are added), the read pattern (how logs are queried), and the retention policies (how long logs are stored). Given these requirements, an LSM (Log-Structured Merge) tree architecture is highly suitable. Below, I’ll explain why this approach is effective and walk through the detailed design.

Firstly, let’s understand the core differences between a B-tree and an LSM. A B-tree is designed for random read access, which isn’t ideal for our append-only write pattern since it requires frequent rebalancing and can lead to inefficient writes. On the other hand, an LSM-Tree is optimized for write-heavy workloads with infrequent reads, making it a better fit for log storage.

TL;DR

I would choose an LSM-tree design because it efficiently handles append-only writes, supports efficient read operations through internal and external sorts, and allows flexible retention policies. The key idea is to use a combination of memory-mapped files and on-disk sorted runs to achieve high throughput and durability with minimal overhead.

#### Triage

The question asks for the choice between an LSM-tree, B-tree, or a segment-file design like Kafka, justifying based on write and read patterns as well as retention policies. The key factors are:

Write Pattern: Append-only writes.
Read Pattern: Infrequent reads of recent logs with occasional full scans.
Retention Policy: Flexible duration for storing events.

#### First Principles Solution
1. Write Pattern: Append-Only Writes

In an append-only log, we need to ensure that new entries are written at the end of a file or files. This ensures that writes are efficient and can be optimized using write-ahead logging (WAL) techniques. LSM-trees achieve this by writing new data directly into memory-mapped files or temporary locations.

2. Read Pattern: Efficient Reads

For reading, we need to support both simple lookups of recent events and full scans over a large set of events. An LSM-tree efficiently handles these patterns through its tiered storage model:

Memory Tier: Stores recently written data in memory for fast access.
Sorted Runs on Disk: Older data is stored as sorted runs, allowing efficient range queries.
3. Retention Policy: Flexible Durability

Retention policies are critical for managing the lifecycle of logs. LSM-trees can be configured to periodically flush memory-mapped files to disk and merge smaller runs into larger ones, ensuring that old data is eventually removed or compressed. This process can be tuned based on the specific retention requirements.

#### Implementation
Details
Memory Tier: Use a cache (like TCMalloc) to store recently written events. This ensures fast access for frequently queried logs.
Sorted Runs: Append new events directly into memory-mapped files, which are later flushed to disk and sorted.
Compaction: Periodically merge smaller runs of data into larger ones to reduce the number of files and improve read performance.
Comparing with B-tree and Segment-File Design
B-tree

#### Pros
Efficient for random access.

#### Cons
Frequent rebalancing during writes, inefficient for append-only patterns. Not suitable due to high write overhead.
Segment-File (Kafka-like)

#### Pros
Append-only writes are handled efficiently, partitioning supports parallel reads and writes.

#### Cons
No internal sorting, less efficient for full scan operations; requires complex topic management and broker coordination.

#### Closing Summary

An LSM-tree is the best choice for an append-only event log storage engine. It provides high write throughput, efficient read patterns through internal and external sorts, and flexible retention policies by managing memory-mapped files and on-disk sorted runs. The architecture ensures durability and scalability, making it ideal for use cases where logs are frequently written but rarely read.

#### Common Mistakes/Follow-Ups
Failing to Recognize: Not identifying the core write pattern as append-only.
Overlooking Retention: Ignoring the importance of retention policies in determining data lifecycle management.
Underestimating Read Patterns: Failing to account for frequent read and occasional full scan operations.

**Comparison:**

```
Memory Tier	Disk Runs Tier
Fast access, caches recent events	Sorted and compacted runs for efficient read operations
```

**Code:**

```python
# Example of initializing a simple LSM-tree log storage system
class LSMTreeLog:
    def __init__(self, memory_capacity, compaction_threshold):
        self.memory_cache = {}
        self.disk_runs = []
        self.memory_capacity = memory_capacity
        self.compaction_threshold = compaction_threshold

    def append(self, event):
        # Write to memory cache
        self.memory_cache[event.id] = event
        if len(self.memory_cache) > self.memory_capacity:
            # Flush cache to disk runs and compact
            self.flush_to_disk()
            self.compact_runs()

    def flush_to_disk(self):
        for event in self.memory_cache.values():
            self.disk_runs.append(event)
        self.memory_cache.clear()

    def compact_runs(self):
        if len(self.disk_runs) > self.compaction_threshold:
            # Merge smaller runs into larger ones
            sorted_runs = sorted(self.disk_runs, key=lambda x: x.timestamp)
            self.disk_runs = [sorted_runs[i:i+10] for i in range(0, len(sorted_runs), 10)]
```

**Example input / output:**

```text
Input:
- Event to append: { "id": "1234", "timestamp": "2023-10-01T12:00:00Z", "data": "First event" }

Output (after appending and compaction):
Memory Cache: {}
Disk Runs: [
    [ Event id=1234, timestamp="2023-10-01T12:00:00Z", data="First event" ],
    ...
]
```

**Data:**

```json
{
  "memory_capacity": 1000,
  "compaction_threshold": 5000,
  "event": {
    "id": "1234",
    "timestamp": "2023-10-01T12:00:00Z",
    "data": "Event details"
  }
}
```

**Flow:**

1. Yes
2. No
3. Found
4. Not Found
5. Yes
6. No
7. New Event
8. Memory Cache Full?
9. Flush to Disk
10. Compact Runs
11. Store in Memory Cache
12. Read Request
13. Check Memory Cache
14. Return Event from Cache
15. Search Disk Runs
16. Disk Run Full?
17. Re-Compact Runs
18. Retrieve Event and Return
19. Retrieve Event and Return
20. Return Events from CompactRuns

