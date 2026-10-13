---
type: "practice"
difficulty: "Deep"
---

# How do you decide between materialized pre-aggregated rollups versus on-demand query for a high-cardinality observability workload (per-endpoint per-region latency)? Map the storage/compute/freshness triangle and how it shifts at 100x cardinality for AIOps.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Rollups vs On-demand Queries](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Rollups_vs_On-demand_Queries.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing data architectures for high-cardinality observability workloads, such as per-endpoint-per-region latency monitoring in AIOps, the decision between materialized pre-aggregated rollups and on-demand query processing hinges heavily on balancing storage, compute, and freshness requirements. The key is to map these factors using a conceptual triangle, where each side represents one of the three critical dimensions: storage cost, real-time query performance, and data freshness. At 100x cardinality, this triangle shifts dramatically, pushing us towards more sophisticated strategies.

At a high level, pre-aggregated rollups sacrifice freshness for reduced storage costs and faster querying by storing summarized versions of data. On-demand queries offer maximum freshness but come with higher computational overhead. For AIOps use cases like endpoint-region latency, the decision requires careful consideration of these trade-offs.

**Comparison:**

```
Factor	Pre-aggregated Rollups	On-Demand Queries
Storage Cost	Low (aggregates data)	High (stores raw data)
Query Performance	Fast (pre-computed summaries)	Slow (full scan required)
Freshness	Low (summarized older data)	High (latest individual records)
```

**Code:**

```python
# Example Python pseudocode to illustrate query vs pre-aggregation logic

def decide_aggregation策略(endpoint_count):
    if endpoint_count < 100:
        return "on-demand"
    else:
        return "pre-aggregated"

endpoint_count = 10_000  # Hypothetical number of endpoints
aggregation_strategy = decide_aggregation策略(endpoint_count)
print(f"Aggregation strategy for {endpoint_count} endpoints: {aggregation_strategy}")
```

**Example input / output:**

```

Input: A dataset with per-endpoint-per-region latency measurements.

PLAINTEXT
Copy
{
  "endpoints": [
    {"endpoint_id": 1, "region": "US-East-1", "timestamp": "2023-10-01T09:00Z", "latency_ms": 30},
    {"endpoint_id": 1, "region": "US-West-2", "timestamp": "2023-10-01T09:00Z", "latency_ms": 45}
  ]
}

Output for pre-aggregated strategy:

PLAINTEXT
Copy
{
  "regions": {
    "US-East-1": {"average_latency_ms": 30},
    "US-West-2": {"average_latency_ms": 45}
  }
}

Output for on-demand query strategy:

PLAINTEXT
Copy
{
  "regions": [
    {"region": "US-East-1", "latency_ms": [30]},
    {"region": "US-West-2", "latency_ms": [45]}
  ]
}
```

**Data:**

```json
{
  "endpoints": [
    {"region": "US-East-1", "latency_ms": 30},
    {"region": "US-West-2", "latency_ms": 45}
  ]
}
```

**Flow:**

1. On-Demand Query Flow
2. Pre-aggregation Flow
3. High Cardinality Data
4. Pre-aggregated Rollups
5. Is the query pattern simple?
6. Yes
7. No
8. On-Demand Queries
9. Balanced Storage/Compute
10. Balanced Freshness/Storage
11. Aggregate and storesummarized data
12. Query pre-aggregatedsummaries quickly
13. Store raw data
14. Process queries on-the-fly withfull scan

