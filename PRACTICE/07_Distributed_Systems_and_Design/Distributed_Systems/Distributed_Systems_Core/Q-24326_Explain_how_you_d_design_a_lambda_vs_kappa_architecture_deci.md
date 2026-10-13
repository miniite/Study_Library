---
type: "practice"
difficulty: "Expert"
---

# Explain how you'd design a lambda vs kappa architecture decision for a metrics platform where you need both real-time dashboards and exact historical recompute. Where does maintaining two code paths earn its keep versus streaming-first?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Choosing an Event Backbone](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Choosing_an_Event_Backbone.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a lambda vs kappa architecture decision for a metrics platform that requires both real-time dashboards and exact historical recompute, you must first understand the key differences between these architectures. A lambda architecture processes data in parallel using batch and stream processing, allowing for real-time queries but potentially leading to discrepancies due to delays in full recomputation. In contrast, a kappa architecture is based on a streaming-first approach that processes all data through streams only, ensuring consistency at the cost of increased complexity.

The decision between these architectures hinges on several factors: real-time requirements, historical accuracy needs, and operational simplicity. Maintaining two code paths (lambda) versus a single stream processing path (kappa) has trade-offs that must be carefully weighed.

**Code:**

```python
# Example Lambda Code Path
def lambda_process(data):
    # Process data for real-time dashboard updates
    pass

# Example Kappa Code Path
def stream_process(data):
    # Process all data through a single stream, ensuring historical accuracy
    pass
```

**Example input / output:**

| Input Type | Lambda Architecture | Kappa Architecture |
|------------|---------------------|--------------------|
| Real-time Dashboards | Supported, with potential data discrepancies | Supported, but may have slight delays due to stream processing |
| Exact Historical Recompute | Not supported, historical data is not recomputed in real-time | Supported, ensuring full recomputation at all times |

**Data:**

```yaml
metrics_platform_architecture:
  - architecture: lambda
    features:
      - real_time_dashboard_updates: true
      - exact_historical_recompute: false
  - architecture: kappa
    features:
      - real_time_dashboard_updates: true
      - exact_historical_recompute: true
```

