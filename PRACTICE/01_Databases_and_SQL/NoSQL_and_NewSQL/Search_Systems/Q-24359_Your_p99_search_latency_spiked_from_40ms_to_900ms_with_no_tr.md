---
type: "practice"
difficulty: "Deep"
---

# Your p99 search latency spiked from 40ms to 900ms with no traffic change. Walk me through the systematic investigation using the slow log, hot threads, and node stats to find the culprit.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Search Index Internals and Mappings](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When faced with a sudden spike in p99 search latency from 40ms to 900ms without any traffic change, our primary goal is to systematically investigate potential causes within Elasticsearch and its underlying infrastructure. We'll use three key tools: slow logs, hot threads, and node stats. Each tool provides a different perspective on the system's behavior and helps us pinpoint the issue.

First, let's understand what these tools are and why they are crucial:

Slow Logs: Capture queries that exceed a certain latency threshold. They provide direct evidence of slow-running queries.
Hot Threads: Expose threads that are consuming a lot of CPU or blocking resources, indicating potential bottlenecks.
Node Stats: Offer detailed insights into the node's overall health, including memory usage, disk I/O, and network throughput.
Step-by-Step Investigation

Review Slow Logs

Start by analyzing the slow logs to identify any queries that might be responsible for the increased latency.
BASH
# Example command (adjust based on your Elasticsearch version)
curl -XGET "localhost:9200/_slowlog/explain?level=total_time[0.1]"
Look for queries with a total_time that significantly exceeds the threshold, and note their execution time and impact.

Analyze Hot Threads

Use tools like Kibana or curl to access Elasticsearch's _cat/threads API.
BASH
# Example command
curl -XGET "localhost:9200/_cat/threads?v&h=node,name,pid,thread_state"
Focus on threads that are RUNNABLE and consuming a lot of CPU. These could indicate heavy query processing or other issues.

Check Node Stats

Access the _nodes/stats API to get comprehensive metrics.
BASH
# Example command
curl -XGET "localhost:9200/_nodes/stats"
Pay attention to metrics like jvm.memory, fs.stats, and indices.search. High memory usage, disk I/O wait, or search latency can indicate issues.

Cross-Refer Data

Cross-reference the findings from slow logs with hot threads and node stats.
Identify patterns such as specific queries causing high CPU usage, memory pressure, or disk contention.

Isolate and Test

Once you have identified a potential culprit (e.g., a problematic query), isolate it by temporarily removing or optimizing that query.
SQL
# Example of an optimized query
SELECT * FROM index WHERE timestamp > '2023-10-01' AND term LIKE '%searchterm%'
Monitor the impact on latency and performance to confirm your findings.

#### Closing Summary
+ Common Mistakes/Follow-Ups

By systematically using slow logs, hot threads, and node stats, we can effectively pinpoint the cause of increased search latency. Key steps include:

Identifying slow queries from slow logs.
Analyzing thread states for CPU-intensive operations.
Monitoring overall system health via node stats.

#### Common mistakes
to avoid include ignoring cross-referencing data between tools or not thoroughly isolating and testing potential causes before concluding an issue. Follow-up questions could involve discussing long-term strategies to prevent such spikes, like query optimization practices or tuning Elasticsearch parameters.

**Comparison:**

| Node | Attribute        | Value          |
|------|------------------|----------------|
| A5B6C7D8E9F0 | Timestamp (ms)    | 1634246400000 |
|         | Host             | 10.0.0.1       |
|         | Open FDs         | 256            |
|         | Max FDs          | 1024           |
|         | CPU Usage (%)    | 7.8            |

**Code:**

```bash
# Example of curl commands for slow logs, hot threads, node stats
curl -XGET "localhost:9200/_slowlog/explain?level=total_time[0.1]"
curl -XGET "localhost:9200/_cat/threads?v&h=node,name,pid,thread_state"
curl -XGET "localhost:9200/_nodes/stats"
```

**Example input / output:**

```text
{
  "slow_logs": [
    {
      "took": 895,
      "query": "SELECT * FROM index WHERE timestamp > '2023-10-01' AND term LIKE '%searchterm%'",
      "node_name": "node-1"
    }
  ],
  "hot_threads": [
    {
      "thread_name": "transport_action_send_request[9.3.0][10.0.0.1:9200][p][n]",
      "state": "RUNNABLE",
      "cpu_usage": 78%
    }
  ],
  "node_stats": {
    "total_nodes": 1,
    "memory_usage": {
      "heap_used_in_bytes": 295560384,
      "heap_max_in_bytes": 778576320
    },
    "disk_io_wait_percent": 20%
  }
}
```

**Data:**

```json
{
  "_nodes": {
    "total": 1,
    "successful": 1,
    "failed": 0,
    "cluster_name": "my_cluster",
    "nodes": {
      "A5B6C7D8E9F0": {
        "timestamp_in_millis": 1634246400000,
        "name": "node-1",
        "transport_address": "10.0.0.1:9300",
        "host": "10.0.0.1",
        "ip": "10.0.0.1:9200",
        "attributes": {
          "ml.machine_memory": "4275686400"
        },
        "os": {
          "available_processors": 4,
          "mem": {
            "total_in_bytes": 4379345600,
            "free_in_bytes": 154698240,
            "used_in_bytes": 4224647360
          }
        },
        "process": {
          "timestamp_in_millis": 1634246400000,
          "open_file_descriptors": 256,
          "max_file_descriptors": 1024,
          "cpu": {
            "percent": 7.8
          }
        },
        "jvm": {
          "timestamp_in_millis": 1634246400000,
          "mem": {
            "heap_used_in_bytes": 295560384,
            "heap_max_in_bytes": 778576320
          },
          "gc_collection_count": 0,
          "thread_count": 16,
          "peak_thread_count": 16
        }
      }
    }
  }
}
```

**Flow:**

1. Start
2. Review Slow Logs
3. Identify Slow Queries
4. Analyze Hot Threads
5. Find CPU Intensive Operations
6. Check Node Stats
7. Monitor Memory, Disk I/O
8. Cross-Reference Findings
9. Test and Optimize
10. Confirm Fixes

