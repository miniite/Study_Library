---
type: "practice"
difficulty: "Expert"
---

# Your team wants native histograms in Prometheus for lower-cardinality latency tracking; what changes for storage, queries, and your downstream anomaly models?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Prometheus TSDB and Scrape Health](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Time_Series_Databases/Prometheus_TSDB_and_Scrape_Health.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To support native histograms in Prometheus for low cardinality latency tracking, several changes need to be considered across storage, querying, and downstream anomaly models. This solution leverages Prometheus' existing time-series database capabilities while introducing histogram metrics.

#### Triaging the Question

The question asks about integrating native histogram support into Prometheus for more efficient and accurate monitoring of lower-cardinality latency data. The core idea is to understand how this change impacts storage efficiency, query performance, and anomaly detection models. We'll assume that your team has a basic understanding of Prometheus' current architecture and metrics.

#### First-Principles Solution

Storage Changes:

Histograms as Time-Series Metrics: Histograms are typically represented as multiple time-series metrics (e.g., histogram_bucket for each bucket value). This increases the number of series stored but provides more detailed data.
Compression and Retention Policies: To mitigate storage overhead, consider implementing effective compression strategies and retention policies. For example, binning histograms into fewer buckets can reduce the storage footprint without losing too much detail.

Querying Changes:

Complexity of Queries: Histogram queries become more complex as they involve joining multiple time-series metrics (buckets) to calculate aggregate statistics like mean, median, and percentiles.
Efficient Query Optimization: Use Prometheus' built-in query optimizations such as histogram_quantile functions for efficient calculation of percentile values. Custom query optimization techniques might be necessary if the increased complexity impacts performance.

Downstream Anomaly Models:

Model Adaptation: Existing anomaly detection models may need to adapt to handle more granular data from histograms. This could involve retraining models with new features derived from histogram buckets.
Real-Time Monitoring: Ensure that real-time monitoring systems can process and visualize the additional metrics without performance degradation.

#### Closing Summary
and Common Mistakes

Integrating native histograms into Prometheus requires careful consideration of storage, query optimization, and model adaptation. Key challenges include managing increased storage requirements, optimizing complex queries, and ensuring robust anomaly detection models can handle the new data format. Common mistakes include over-optimizing queries at the cost of query complexity or underestimating the impact on storage management.

#### Follow-Ups
How would you balance the number of histogram buckets to minimize storage overhead while maintaining necessary accuracy?
Can you suggest any specific techniques for optimizing Prometheus queries when dealing with histograms?
What are potential issues in adapting anomaly detection models, and how might these be addressed?

**Comparison:**

```
Metric Type	Time-Series Metrics
Histogram	histogram_bucket
Count	http_request_duration_seconds_count
Sum	http_request_duration_seconds_sum
Retention Policy	Description
duration	7 days
interval	1 hour
shard_duration	24 hours
max_shards_per_series	30
max_shards_per_node	600
Compression Config	Description
min_block_size	5 minutes
compression_level	Level 5
```

**Code:**

```
PROMETHEUS
Copy
# Example histogram metrics
histogram_bucket{le="0.1", job="service1"} 0
histogram_bucket{le="0.25", job="service1"} 10
histogram_bucket{le="1", job="service1"} 90

# Query for the 95th percentile latency
histogram_quantile(0.95, sum by (job) (rate(http_request_duration_seconds_sum[1m])))
```

**Example input / output:**

```text
Input:
- http_request_duration_seconds_sum, http_request_duration_seconds_count, and histogram_bucket metrics for service1.

Output:
- 95th percentile latency for service1 over the last minute.
```

**Data:**

```yaml
retention_policy:
  - name: default
    duration: 7d
    interval: 1h
    shard_duration: 24h
    max_shards_per_series: 30
    max_shards_per_node: 600

compression_config:
  - min_block_size: 5m
    compression_level: 5
```

**Flow:**

1. Yes
2. No
3. Query Histogram Metrics
4. Is Query Complex?
5. Optimize Query for Efficiency
6. Process Data
7. Generate Anomaly Signals
8. Store Histograms withCompression
9. Manage Retention Policies

