---
type: "practice"
difficulty: "Deep"
---

# Search p99 spiked after a big reindex while p50 stayed flat — walk through how you'd diagnose whether it's segment merges GC cold caches or a few heavy queries and how you'd protect tail latency.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Backend Design](<../../../../07_Distributed_Systems_and_Design/System_Design/High_Level_Design/Search_Backend_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To diagnose why the 99th percentile (p99) of search latency spiked after a big reindex while the 50th percentile (p50) stayed flat, we need to investigate potential causes such as segment merges, garbage collection cold caches, and heavy queries. The key is understanding how these factors impact different parts of the system and identifying which ones are causing disproportionate delays in tail latencies.

#### Triaging the Question

The interviewer is probing your ability to diagnose complex performance issues in a distributed search engine. Specifically, you need to:

Understand the difference between p99 and p50 – p99 represents the latency experienced by 1% of users, which is often more relevant for tail latencies.
Identify potential causes – Segment merges (the process of merging smaller segments into larger ones), garbage collection cold caches, and heavy queries are likely culprits.
Design a diagnostic approach – This involves collecting metrics, analyzing logs, and possibly implementing temporary workarounds to isolate the issue.

#### Assumptions
We have a distributed search engine with multiple nodes.
There was recently a large reindex operation that significantly changed the segment structure.
The system is using garbage collection (GC) to manage memory usage.
Heavy queries are those that consume more resources than normal and could cause delays.

#### First-Principles Solution
1. Understand Segment Merges

Segment merges can lead to increased tail latencies because the process of merging segments involves reading data from disk, which is inherently slower compared to in-memory operations. This might impact p99 latency disproportionately if a large number of small segments need to be merged.

Diagnosis: Check the segment merge logs and metrics for any unusual spikes.
Impact on P50 vs P99: Segment merges generally do not affect p50 significantly because they are a background process that operates in parallel. However, they can cause significant delays in tail latencies due to increased I/O operations.
2. Analyze Garbage Collection Cold Caches

Garbage collection (GC) pauses can lead to spikes in latency, especially if the GC cycle is triggered unexpectedly or due to cold caches. Cold caches occur when a node has not been active for some time and needs to fetch data from disk, which can be much slower.

Diagnosis: Monitor GC pause times and cache hit rates. Look for any correlation between these metrics and the spike in p99 latency.
Impact on P50 vs P99: Cold caches tend to affect all nodes equally during a reindex operation. However, if some nodes are cold-cached more frequently than others due to uneven segment distribution or query patterns, it can lead to higher tail latencies.
3. Investigate Heavy Queries

Heavy queries are those that consume significantly more resources than normal and could cause delays in processing other queries, especially during a reindex operation when the system is under stress.

Diagnosis: Analyze logs and metrics for any heavy queries that were running at the time of the latency spike.
Impact on P50 vs P99: Heavy queries can impact both p50 and p99 but often have a more significant effect on tail latencies because they block other queries from being processed.
Diagnostic Steps:

Collect Metrics:

Segment merge statistics
GC pause times and cache hit rates
Query execution time histograms

Analyze Logs:

Search query logs to identify any heavy queries
GC logs for any unusual activity

Temporarily Implement Workarounds:

Limit the number of concurrent segment merges during reindex operations.
Use query throttling or prioritization to manage resource consumption.

#### Closing Summary
and Common Mistakes/Follow-Ups:

To effectively diagnose the issue, it's crucial to understand how different components interact under stress. Segment merges can cause significant tail latencies due to increased I/O, while GC cold caches and heavy queries can contribute to overall latency spikes. By monitoring relevant metrics and logs, you can identify the root cause and implement appropriate workarounds.

#### Common Mistakes

Focusing only on average performance metrics (e.g., p50) instead of tail latencies.
Ignoring the impact of background processes like segment merges and GC pauses.
Overlooking the role of heavy queries in causing disproportionate delays.

#### Follow-Ups

How would you ensure that your solution is scalable for future reindex operations?
What steps would you take to prevent similar issues from occurring in the future?

**Comparison:**

```
Node	Merge Count (Last Hour)	Total Time(ms)	Merge Rate (s^-1)
Node1	500	25000	2.0
Node2	300	18000	1.7
```

**Code:**

```python
# Example code snippet for monitoring segment merge statistics
def monitor_segment_merges():
    # Simulate fetching segment merge stats
    merges = {
        "node1": {"merge_count": 500, "total_time_ms": 25000},
        "node2": {"merge_count": 300, "total_time_ms": 18000}
    }
    for node, stats in merges.items():
        merge_rate = stats["merge_count"] / (stats["total_time_ms"] / 1000)
        print(f"Node {node}: Merge rate - {merge_rate} merges/s")
```

**Example input / output:**

```text
Input:
- Monitor segment merge statistics for node1 and node2.
- Collect GC pause times and cache hit rates.
- Analyze query logs to identify heavy queries.

Output:
- Node1: Merge rate - 5 merges/s, P99 latency increased by 40%
- Node2: Merge rate - 3 merges/s, no significant change in latency
- No heavy queries identified during the spike

Conclusion:
Segment merge activity on node1 is causing a spike in tail latencies.
```

**Data:**

```json
{
  "segment_merge_stats": {
    "node1": {"merge_count": 500, "total_time_ms": 25000},
    "node2": {"merge_count": 300, "total_time_ms": 18000}
  },
  "gc_logs": [
    {"time": "2023-10-01T14:30:00Z", "pause_duration_ms": 500},
    {"time": "2023-10-01T14:31:00Z", "pause_duration_ms": 600}
  ],
  "query_stats": [
    {"query_id": "q123", "exec_time_ms": 150},
    {"query_id": "q456", "exec_time_ms": 300}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Monitor Segment Merges
4. Check for Unusual MergeActivity
5. Is p99 Increased?
6. Investigate GC Cold Cachesand Heavy Queries
7. Reindex Operation CompletedSuccessfully
8. Analyze Logs, ImplementWorkarounds
9. Verify Latency Improvements

