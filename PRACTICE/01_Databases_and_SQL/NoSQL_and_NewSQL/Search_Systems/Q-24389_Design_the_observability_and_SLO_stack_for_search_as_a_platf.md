---
type: "practice"
difficulty: "Deep"
---

# Design the observability and SLO stack for search as a platform so tenants can self-diagnose their own relevance and latency without paging your team. What metrics do you expose per-tenant, and how do you attribute cluster-level pain to specific tenants?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Search Platform Scaling](<../../../../01_Databases_and_SQL/NoSQL_and_NewSQL/Search_Systems/Search_Index_Internals_and_Mappings/Search_Platform_Scaling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an observability and Service Level Objective (SLO) stack for Elasticsearch in a platform-as-a-service model, ensuring tenants can self-diagnose their own relevance and latency issues, we need to focus on exposing detailed metrics per-tenant while attributing cluster-level performance issues back to specific users. The solution involves leveraging Elasticsearch’s rich metric exposure capabilities, implementing tenant-specific dashboards, and using a combination of aggregation, anomaly detection, and root cause analysis techniques.

The primary goal is to empower tenants with the necessary insights so they can identify and resolve their own issues without needing constant intervention from your team. This requires a layered approach: first, defining the key metrics that matter for search performance; second, setting up tenant-specific alerting and diagnostics tools; and third, ensuring these tools are easy to use and interpret.

**Comparison:**

```
Tenant ID	Relevance Score (avg)	Latency Avg (ms)
t1	8.5	30
t2	7.3	45
```

**Code:**

```python
# Example of exposing metrics in Elasticsearch via API or Kibana dashboard
from elasticsearch import Elasticsearch

es = Elasticsearch([{"host": "localhost", "port": 9200}])

# Query for tenant-specific metrics
response = es.search(index="search-tenant-metrics-*", body={
    "size": 0,
    "aggs": {
        "by_tenant": {
            "terms": {"field": "tenant_id"},
            "aggs": {
                "relevance_score": {"avg": {"field": "relevance_score"}},
                "latency_avg": {"avg": {"field": "response_time_ms"}}
            }
        }
    }
})
```

**Example input / output:**

```

Input: A request to fetch tenant-specific metrics from Elasticsearch.

Output:

JSON
Copy
{
  "buckets": [
    {
      "key": "t1",
      "relevance_score": {"value": 8.5},
      "latency_avg": {"value": 30}
    },
    {
      "key": "t2",
      "relevance_score": {"value": 7.3},
      "latency_avg": {"value": 45}
    }
  ]
}
```

**Data:**

```json
{
  "tenant_id": ["t1", "t2"],
  "relevance_score": [8.5, 7.3],
  "latency_avg": [30, 45]
}
```

**Flow:**

1. Metrics Collection
2. Aggregation by Tenant
3. Tenant-Specific Dashboard
4. Anomaly Detection & Alerting
5. Root Cause Analysis

