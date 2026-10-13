---
type: "practice"
difficulty: "Deep"
---

# Your autoscaler scales GPU inference on CPU utilization, but the workload is actually KV-cache-pressure bound, so it scales at the wrong time. Explain why this signal is wrong and what metric (queue depth, batch latency, cache occupancy) you'd scale on instead.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Workload Isolation on Shared Clusters](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Mixed_Version_Fleets/Workload_Isolation_on_Shared_Clusters.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where an autoscaler scaling based on CPU utilization misinterprets the workload's true bottleneck as being KV-cache-pressure bound, we need to understand that the CPU utilization is not a direct reflection of the actual performance bottleneck in GPU inference tasks. The key metric to consider here is cache occupancy or queue depth, which more accurately indicates when the system is under stress.

In summary, scaling based on CPU usage can lead to suboptimal resource allocation because it does not account for the true nature of the workload's constraints. Instead, we should scale our autoscaler to monitor and react to metrics that directly reflect the real performance issues—specifically cache occupancy or queue depth.

**Comparison:**

```
Metric Type	Description	Why It Matters
CPU Utilization	Measures how busy the CPUs are	Misleading as a sole metric for GPU inference tasks
Cache Occupancy	Reflects the pressure on cache systems	Directly indicates when performance is being impacted
```

**Code:**

```python
# Example code snippet to check a hypothetical autoscaler configuration
class Autoscaler:
    def __init__(self, max_gpus=4):
        self.max_gpus = max_gpus

    def scale(self, metric_type='cpu_utilization', threshold=80):
        if metric_type == 'cpu_utilization':
            # This is the current naive implementation based on CPU usage
            pass  # Placeholder for actual logic
        elif metric_type == 'cache_occupancy':
            # Implement scaling based on cache occupancy
            pass  # Placeholder for actual logic

# Example usage with cache_occupancy as the new metric
autoscaler = Autoscaler()
autoscaler.scale(metric_type='cache_occupancy', threshold=80)
```

**Example input / output:**

```text
| Input Metric Type | Input Threshold | Output Scaling Decision |
|-------------------|----------------|------------------------|
| cpu_utilization   | 80             | No scaling needed       |
| cache_occupancy   | 80             | Scale up GPUs           |

For the `cpu_utilization` metric, if the current value is below the threshold (75 < 80), no action is taken. However, for `cache_occupancy`, when it hits 90 and exceeds the threshold of 80, scaling would be triggered.
```

**Data:**

```json
{
  "metrics": {
    "cpu_utilization": {
      "current_value": 75,
      "threshold": 80
    },
    "cache_occupancy": {
      "current_value": 90,
      "threshold": 80
    }
  }
}
```

**Flow:**

1. No
2. Yes
3. Detect Cache Occupancy
4. Occupancy < Threshold?
5. Scale Down GPUs
6. Scale Up GPUs

