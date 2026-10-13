---
type: "practice"
difficulty: "Intermediate"
---

# You see high CPU from checksum verification on every block read in your storage engine. Explain when per-block checksums (CRC32C / xxHash) are worth the CPU, and how you'd decide to relax them for a read-hot path.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Storage in LSM Engines](<../../../../01_Databases_and_SQL/Database_Internals/Storage_Engines/Tiered_Storage_in_LSM_Engines/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Per-block checksum verification can be beneficial in scenarios where data integrity is critical or frequent disk corruption is suspected. However, for high-read-load paths (e.g., cache hits), these verifications might introduce unnecessary overhead. To balance this trade-off, one must assess the specific workload and potential risks.

#### Triaging the Question

The question revolves around understanding when to use per-block checksums in a storage engine, which helps ensure data integrity by verifying each block read from disk. The task is to determine whether these checks are necessary for all reads or if they can be relaxed to improve performance on high-read-load paths. This involves analyzing the impact of CPU overhead and balancing it against potential risks.

#### Assumptions
We have a storage engine with frequent read operations.
There's a trade-off between ensuring data integrity through per-block checksums and reducing CPU overhead for high-read-load paths.
The system operates in an environment where disk corruption might occur, but the frequency of such events is unknown or potentially low.

#### Solution

Understanding Per-Block Checksum Verification:

CRC32C / xxHash: These are commonly used checksums that help verify data integrity by generating a hash value for each block read from storage.
CPU Overhead: Each checksum computation introduces some CPU overhead, especially if this operation is performed on every single block.

Use Cases Where Checksums Are Beneficial:

Data Integrity Critical Scenarios: In mission-critical applications where data integrity must be guaranteed at all costs (e.g., financial transactions, medical records), per-block checksums are essential.
Frequent Disk Corruption Suspected: If there is a history of frequent disk corruption or hardware issues, enabling per-block checksums can help catch these errors early.

High-Read-Load Path Considerations:

Cache Hits vs Misses: Cache hits dominate the read path in most systems, where data retrieval is fast and does not require additional overhead.
CPU Utilization Analysis: Monitoring CPU utilization during high-read-load periods can help identify if checksum verification is causing significant bottlenecks.

Deciding to Relax Checksums:

Performance Metrics: Collect metrics on read performance, including the number of cache hits and misses, and the overall impact on CPU usage.
Risk Assessment: Evaluate the risk of data corruption without checksums. If the probability of disk corruption is low or manageable through other mechanisms (e.g., redundancy), relaxing checksums can be beneficial.

#### Implementation
Strategy:

Conditional Checksum Verification: Implement a mechanism to skip per-block checksum verification on cache hits but perform it on all reads from non-cache sources.
Dynamic Adjustment: Allow the system to dynamically adjust its strategy based on real-time metrics and workload analysis.
Testing and Validation: Thoroughly test any changes in a staging environment before rolling them out to production.

#### Closing Summary

In summary, per-block checksums are essential for maintaining data integrity but can introduce significant CPU overhead. For high-read-load paths where cache hits dominate, relaxing these checks can optimize performance without compromising critical data integrity requirements. The key is to understand the specific workload and potential risks, then make informed decisions based on performance metrics and risk assessments.

#### Common Mistakes/Follow-Ups
Overly Broad Relaxation: Failing to consider edge cases where checksums are still needed.
Ignoring Performance Metrics: Not collecting and analyzing relevant data before making changes.
Insufficient Testing: Deploying untested changes directly into production, leading to unexpected issues.

**Comparison:**

```
Block ID	Cache Hit	Checksum Status
12345	false	true
67890	true	-
```

**Code:**

```python
# Example of conditional checksum verification in Python
def read_block(block_id):
    block = get_block_from_cache(block_id)
    if not block:
        # Read from disk and perform full integrity check
        block = read_block_from_disk(block_id)
        compute_checksum(block)  # Compute full checksum for integrity
    else:
        # Skip full checksum for cache hits
        pass
```

**Example input / output:**

```text
Input:
- `read_block(12345)`

Output:
- If cache hit (`cache_hit` is `false`), perform full integrity check and compute checksum.
- If not a cache hit, read from disk and compute full checksum.

### Explanation of the Code:
The example code shows how to conditionally apply per-block checksums. It checks if a block is cached; if not, it reads from disk and computes a full checksum for integrity verification.
```

**Data:**

```json
{
  "block_id": 12345,
  "cache_hit": false,
  "checksum_status": true, // True if full checksum was performed
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Block Read
6. Cache Hit?
7. C
8. Read from Disk
9. Compute Full Checksum?
10. Verify Integrity
11. Use Block Directly

