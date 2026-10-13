---
type: "practice"
difficulty: "Intermediate"
---

# Your API gateway pods on Kubernetes keep getting OOMKilled under a traffic spike while CPU is fine; how do you size memory requests/limits for a proxy that buffers request bodies, and what metrics tell you it's buffering not leaking?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Scaling for Traffic Spikes](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/Gateway_Scaling_for_Traffic_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of API Gateway pods getting OOMKilled due to traffic spikes while CPU usage is fine, we need to ensure proper memory allocation for the proxy that buffers request bodies. The key idea here is understanding how buffering works and monitoring relevant metrics to identify whether the issue lies in buffering or potential leaks.

#### Triage

The question asks about sizing memory requests/limits for a proxy in Kubernetes when encountering OOMKilled errors during traffic spikes. Specifically, it probes our ability to diagnose the root cause (buffering vs. memory leak) and size the resources accordingly. Key assumptions include:

The proxy buffers request bodies.
There are periodic traffic spikes that lead to OOMKilled events.

#### First Principles Solution
Understand Buffering Behavior: A buffering proxy will temporarily store incoming request data in memory until it can process them, which could consume significant amounts of memory during a spike.
Monitor Relevant Metrics:
container_memory_usage_bytes: Measures the actual memory usage by the container.
container_memory_working_set_bytes: Indicates the total committed memory that the container has used, including buffers and cache.
Set Memory Limits Appropriately: Ensure the limits are high enough to handle spikes without OOMKilled errors but low enough to avoid unnecessary resource consumption.
Detailed Solution
Identify Buffering Behavior:
Check if container_memory_working_set_bytes is consistently higher than container_memory_usage_bytes. If so, it suggests that memory buffers (like HTTP request bodies) are being held in memory.
Adjust Memory Limits:
Increase the limits for the container to ensure sufficient memory capacity during spikes.
Ensure the requests value is set appropriately to avoid unnecessary resource consumption.
Monitor and Optimize:
Use Kubernetes metrics exporters like Prometheus to gather detailed metrics.
Implement alerting rules in monitoring tools (e.g., Grafana) to notify when memory usage exceeds certain thresholds.

#### Closing Summary
+ Common Mistakes

To properly size the memory requests/limits for an API Gateway proxy that buffers request bodies, focus on understanding the buffering behavior and using appropriate metrics. Common mistakes include:

Setting too high a limit, leading to unnecessary resource consumption.
Failing to monitor working set bytes, which can indicate actual buffer usage.
Ignoring other potential issues like memory leaks or misconfigured caching mechanisms.

#### Example
Metrics
YAML
apiVersion: v1
resources:
  limits:
    cpu: "500m"
    memory: "2Gi"
  requests:
    cpu: "500m"
    memory: "1Gi"
Self-Check

This solution ensures proper resource management and monitoring for the API Gateway proxy, addressing the OOMKilled issue effectively.

**Comparison:**

```
Metric	Description
container_memory_usage_bytes	Actual memory used by the container.
container_memory_working_set_bytes	Total committed memory, including buffers and cache.
```

**Code:**

```yaml
apiVersion: v1
resources:
  limits:
    cpu: "500m"
    memory: "2Gi"
  requests:
    cpu: "500m"
    memory: "1Gi"
```

**Flow:**

1. Check Buffering Behavior
2. Monitor Metrics
3. Identify High Working SetBytes
4. Adjust Memory Limits
5. Implement Monitoring &Alerting

