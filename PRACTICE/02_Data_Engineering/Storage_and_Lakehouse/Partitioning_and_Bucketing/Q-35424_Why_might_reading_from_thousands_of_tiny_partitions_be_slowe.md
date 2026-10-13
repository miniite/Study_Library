---
type: "practice"
difficulty: "Intermediate"
---

# Why might reading from thousands of tiny partitions be slower than reading from a few larger ones?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Small Files Problem](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Partitioning_and_Bucketing/Small_Files_Problem.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Reading from thousands of tiny partitions can indeed be slower compared to reading from a few larger ones, primarily due to increased overhead from metadata management and disk I/O operations.

#### Triaging the Question

The core issue here is understanding why partitioning granularity impacts read performance in a database system. The key areas we need to explore are:

Metadata Overhead: How many partitions impact the time spent on metadata lookups.
Disk I/O Operations: The number of files involved and their impact on I/O operations.
Cache Performance: How partitioning affects query cache efficiency.

#### First Principles Solution
Metadata Overhead

In most database systems, partitioning involves maintaining a catalog or metadata that tracks the location of each data segment. Reading from thousands of tiny partitions means the system needs to perform more metadata lookups. Each lookup adds overhead due to network latency and processing time. For instance, in Apache Parquet, each directory listing can be considered an additional lookup, which can significantly slow down queries.

Disk I/O Operations

Reading from a large number of files increases the total I/O operations required. Each file read involves seeking the disk head to the correct location, which can be time-consuming. Additionally, if the files are small and scattered, it may lead to more seek times and less efficient use of cache, further slowing down reads.

Cache Performance

Large partitions tend to fit better into the database’s query execution plan cache (like HDFS block caching or Parquet metadata caching). This means that for a given query, fewer cache misses will occur with larger partitions. Smaller partitions increase the likelihood of cache misses, as they may not provide enough data blocks per partition to be effectively cached.

#### Closing Summary

In summary, reading from thousands of tiny partitions can introduce significant overhead due to increased metadata lookups and disk I/O operations. This reduces overall read performance compared to a few larger partitions that are better suited for caching and query optimization.

#### Common Mistakes
and Follow-Ups
Ignoring Metadata Overhead: Many engineers focus solely on file size but overlook the impact of metadata management.
Assuming Uniform Performance: Different databases handle partitioning differently, so assumptions about performance should be validated with specific benchmarks.
Over-partitioning: While smaller partitions can improve query performance by allowing more precise filtering, over-partitioning (too many tiny partitions) can negate these benefits due to increased metadata and I/O overhead.

#### Follow-up
questions might include:

How does partition pruning impact read performance?
What are the best practices for determining the optimal number of partitions?
Can you provide an example of a scenario where over-partitioning would be beneficial?

**Comparison:**

| Partitions | Metadata Lookups (per query) | I/O Operations |
|------------|------------------------------|---------------|
| 100        | 100                          | ~30           |
| 10         | 10                           | ~27           |

**Code:**

```python
# Example code to illustrate metadata overhead in a simplified way
import time

def simulate_metadata_overhead(n_partitions):
    start_time = time.time()

    for _ in range(n_partitions):
        # Simulate a lookup operation, e.g., fetching metadata or file path
        pass

    end_time = time.time()
    return (end_time - start_time) / n_partitions

# Test with different number of partitions
print(simulate_metadata_overhead(100))  # Small partitions
print(simulate_metadata_overhead(10))   # Fewer, larger partitions
```

**Example input / output:**

```text
Input:
- Number of partitions: 1000

Output:
- Time per metadata lookup: ~3ms
- Total time for metadata lookups: 3s (assuming each lookup takes 3ms)

Input:
- Number of partitions: 10

Output:
- Time per metadata lookup: ~27ms
- Total time for metadata lookups: 0.27s (assuming each lookup takes 27ms)
```

**Data:**

```json
{
  "partitions": [
    {"id": 1, "size": 5MB},
    {"id": 2, "size": 5MB},
    ...
    {"id": 1000, "size": 5MB}
  ]
}
```

**Flow:**

1. Start
2. Metadata Overhead?
3. Perform More Lookups
4. Increased I/O Time
5. I/O Operations Increase
6. Reduce Cache Hits

