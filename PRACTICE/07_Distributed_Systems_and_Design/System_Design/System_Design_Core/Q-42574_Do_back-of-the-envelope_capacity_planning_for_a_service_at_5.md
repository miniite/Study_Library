---
type: "practice"
difficulty: "Deep"
---

# Do back-of-the-envelope capacity planning for a service at 50k RPS: estimate DB QPS after caching, connection count, cache memory, and network egress. Where do you add safety margin and why?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Org-wide Backpressure](<../../../../07_Distributed_Systems_and_Design/System_Design/System_Design_Core/Rate_Limiter_Storage/Org-wide_Backpressure.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To perform back-of-the-envelope capacity planning for a service handling 50k RPS (requests per second), we need to estimate key system components such as database queries per second (QPS) after caching, connection count, cache memory usage, and network egress. This analysis involves understanding the request distribution, caching efficiency, and system constraints.

#### Triaging the Question

The interviewer is probing your ability to break down complex service requirements into manageable subsystems, estimate their capacities, and apply safety margins for reliability. The main assumptions are:

Each request can be classified as a read or write operation.
Caching reduces the number of database queries.
Connection pool size affects overall throughput.
Cache memory limits determine how much data can be cached.

#### First Principles Solution
Step 1: Estimate Total Requests

Assume 50k RPS is composed of reads (80%) and writes (20%).

TEXT
Total RPS = 50,000
Reads = 50,000 * 0.8 = 40,000 RPS
Writes = 50,000 * 0.2 = 10,000 RPS
Step 2: Estimate Database QPS after Caching

Assume a hit rate of 70% for caching.

TEXT
Cache Hits (Reads) = 40,000 * 0.7 = 28,000 cached reads per second
DB QPS from cache misses = 40,000 - 28,000 = 12,000 DB QPS for reads
Writes remain unchanged: 10,000 WQPS (Write queries per second)
Step 3: Connection Count and Pool Size

Assume a connection pool size of 50 to handle concurrency.

TEXT
Total QPS = 12,000 + 10,000 = 22,000 QPS

Connection Count = Total QPS / Connections per request ~ 22,000 / 2 (average) ≈ 11,000 connections
Step 4: Cache Memory and Hit Rate

Assume a cache memory of 5GB to store frequently accessed data.

TEXT
Cache Memory = 5 GB
Data per key = Assume an average key size of 32 bytes (key) + 64 bytes (value) = 96 bytes
Total keys in cache ≈ 5 * 10^9 / 96 ≈ 52.1 million keys

Given a hit rate of 70%, we need to ensure the cache can handle at least this many misses.
Step 5: Network Egress and Bandwidth

Assume an average packet size of 1,500 bytes.

TEXT
Writes (Egress) = 10,000 WQPS * 1,500 bytes ≈ 15 MB/s

Reads (Egress) = 28,000 cached reads * 32 bytes ≈ 900 KB/s + un-cached reads * 1,500 bytes

#### Closing Summary
and Safety Margin

Adding a safety margin is crucial to ensure reliability under load. Typically, a 10-20% buffer is used for all components.

DB QPS: Add 10-20% to account for unexpected spikes.
Connections: Ensure enough spare capacity in the connection pool (e.g., 5-10% extra).
Cache Memory: Consider additional overhead and future growth, possibly doubling or tripling current estimates.
Egress Bandwidth: Plan for potential increases in data size due to protocol overheads.

By applying these margins, we ensure the system can handle unexpected surges without failure.

#### Common Mistakes/Follow-ups
Ignoring Write Operations: Focusing only on reads can overlook critical write-heavy scenarios.
Overestimating Caching Efficiency: Real-world hit rates often vary from theoretical estimates.
Neglecting Network Latency: Bandwidth calculations should include latency impacts, especially in distributed systems.

**Comparison:**

```
Resource	Capacity	Safety Margin (%)
DB QPS	12,000	15-24
Connections	11,000	5-10
Cache Memory	5 GB	100-300
Network Egress (Writes)	15 MB/s	10-20
```

**Code:**

```python
# Example code for calculating QPS and connections
def calculate_qps(reads, writes):
    total_rps = reads + writes
    cache_hits = reads * 0.7
    db_queries_per_second = (reads - cache_hits) + writes
    return db_queries_per_second

db_qps = calculate_qps(40_000, 10_000)
print(f"DB QPS after caching: {db_qps}")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Total RPS = 50,000 <br> Read Percentage = 80% <br> Write Percentage = 20% <br> Cache Hit Rate = 70% <br> Cache Memory Size = 5GB | DB QPS after Caching ≈ 12,000 <br> Connection Count ≈ 11,000 <br> Cache Keys ≈ 52.1 million <br> Network Egress (Writes) ≈ 15 MB/s <br> Network Egress (Reads) ≈ 900 KB/s |

**Data:**

```json
{
  "cache": {
    "hit_rate": 70,
    "memory_size_gb": 5
  },
  "requests": {
    "total_rps": 50_000,
    "read_percentage": 80,
    "write_percentage": 20
  }
}
```

**Flow:**

1. Estimate Total Requests
2. Calculate Cache Hits
3. Calculate DB QPS afterCaching
4. Calculate Connection Count
5. Calculate Cache MemoryRequirements
6. Calculate Network EgressBandwidth
7. Apply Safety Margins andFinalize Estimates

