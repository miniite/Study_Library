---
type: "practice"
difficulty: "Deep"
---

# Explain the trade-off between storing pre-computed correlation edges in the lake versus computing them at query time during an incident.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Exactly Once into Observability Lakes](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Observability_Data_Lake/Exactly_Once_into_Observability_Lakes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In AIOps, observability data lakes play a crucial role in aggregating and storing vast amounts of operational data from various sources. The question asks us to weigh the trade-off between pre-computing correlation edges and storing them in the lake versus computing these edges on-the-fly during an incident. This involves balancing factors like performance, storage efficiency, query flexibility, and real-time response times.

To understand this fully, we need to delve into first principles: what are correlation edges? They represent relationships or patterns between different metrics or logs that might indicate a root cause of an issue. Pre-computing these edges means storing them in the data lake for quicker access during incidents, whereas computing them on-the-fly involves calculating them at query time.

#### Triaging the Question

#### Performance Considerations
Storing pre-computed correlation edges can significantly reduce query times and response latency during high-stress scenarios like an incident.
Storage Efficiency: Pre-computing might lead to redundant storage, especially if these relationships are not frequently queried.
Query Flexibility vs. Real-Time Analysis: On-the-fly computation allows for dynamic queries but may introduce latency.

#### First-Principles Solution
1. Pre-Computation Benefits
Reduced Query Latency: By storing pre-computed edges, we can quickly retrieve them during an incident, reducing the time it takes to identify and address issues.
Improved Incident Response: Pre-computed relationships provide a baseline that can be used immediately to understand potential causes of incidents without any delay.
2. On-The-Fly Computation Benefits
Resource Efficiency: Only compute what is needed at query time, potentially saving on storage costs if these relationships are not frequently accessed.
Dynamic Queries: Real-time computation allows for flexible and dynamic queries that can adapt to the evolving nature of data.

#### Example Scenario
s
High-Stress Scenario (Incident): During an incident, pre-computed correlation edges would be invaluable, as they provide immediate insights into potential root causes, allowing teams to respond more quickly.
Low-Stress Scenario: For routine analysis or queries that are less time-sensitive, on-the-fly computation might be sufficient and could save storage costs.
3. Trade-Offs
Latency vs. Storage: Pre-computation reduces query latency but may increase storage requirements.
Cost Efficiency: On-the-fly computation is cost-effective in terms of storage but may introduce additional processing overhead.

#### Closing Summary

In summary, the choice between pre-computed correlation edges and on-the-fly computation depends heavily on the specific use case. For high-stress scenarios like incidents where quick response times are critical, pre-computation offers significant advantages. However, for less time-sensitive queries, on-the-fly computation can be more resource-efficient.

#### Common Mistakes
and Follow-Ups
Overlooking Storage Costs: Teams often focus solely on query performance without considering the long-term storage costs.
Ignoring Dynamic Query Needs: Overstressing the importance of static pre-computation while ignoring dynamic environments that require flexible queries.
Not Considering Data Evolution: Assuming static relationships between data, not accounting for changing patterns over time.

In a more complex AIOps environment, combining both approaches might be ideal. Use pre-computed edges in high-stress scenarios and on-the-fly computation in lower-stress, more dynamic environments to achieve the best balance.

**Comparison:**

| Metric A | Metric B | Pre-Computed Correlation | On-The-Fly Computed Correlation |
|----------|----------|-------------------------|--------------------------------|
| CPU Usage| Disk I/O | -0.85                    | 0.76 (computed at query time)   |
| Disk I/O | Network Latency | 0.92        | -0.43 (computed at query time)  |

**Code:**

```python
# Example of storing pre-computed correlation edges (Python pseudocode)
class CorrelationLake:
    def __init__(self):
        self.edges = {}

    def compute_edge(self, metric1, metric2):
        # Simulate computation logic
        result = f"Correlation between {metric1} and {metric2}"
        self.edges[(metric1, metric2)] = result

    def get_edge(self, metric1, metric2):
        return self.edges.get((metric1, metric2), "Not computed")
```

**Example input / output:**

| Input (Pre-computed Edge) | Output (Correlation Value) |
|---------------------------|----------------------------|
| metric1 -> metric2        | -0.85                      |
| Input (On-the-fly Computation) | Output (Correlation Value) |
|--------------------------------|----------------------------|
| metric1 -> metric2             | 0.76 (computed at query time) |

**Data:**

```json
{
  "metrics": [
    {"id": "metric1", "name": "CPU Usage"},
    {"id": "metric2", "name": "Disk I/O"},
    {"id": "metric3", "name": "Network Latency"}
  ],
  "correlation_edges": [
    {"source_id": "metric1", "target_id": "metric2", "value": -0.85, "timestamp": "2023-10-01T14:00:00Z"},
    {"source_id": "metric2", "target_id": "metric3", "value": 0.92, "timestamp": "2023-10-01T15:00:00Z"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Query for Correlation
5. Is Pre-Computed Available?
6. Retrieve Pre-Computed Edge
7. Compute Edge on-the-Fly
8. Return Correlation Value
9. Compute Correlation Value
10. End

