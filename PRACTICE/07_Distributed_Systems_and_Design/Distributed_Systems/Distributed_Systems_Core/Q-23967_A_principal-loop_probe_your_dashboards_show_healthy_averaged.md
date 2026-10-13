---
type: "practice"
difficulty: "Deep"
---

# A principal-loop probe: your dashboards show healthy averaged latency but a specific customer segment is silently degraded and the AIOps system never flagged it. Explain why aggregate metrics and coordinated omission hide per-segment tail behavior and how you'd instrument distributed monitoring to surface it.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Aggregation Hiding Tail Problems](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/Aggregation_Hiding_Tail_Problems.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In distributed systems, we often rely on aggregated metrics like average latency to gauge system health. However, these can mask significant issues within specific segments of users or traffic, as seen in this scenario where a particular customer segment is experiencing degraded performance while overall averages remain healthy. This issue is primarily due to coordinated omission and the limitations of aggregated metrics.

Firstly, let's understand why aggregate metrics might be misleading. Aggregated latency metrics typically represent the average response time across all requests processed by a system. While useful for high-level insights, they can hide tail behavior—outliers that significantly affect user experience or business-critical operations. For instance, if 95% of your requests are fast but 5% take an exceptionally long time, the average might still look good, obscuring the degradation affecting a critical segment.

Coordinated omission is another reason for this hidden performance issue. In distributed systems, especially those with multiple layers or nodes, operations can be executed in a coordinated manner that masks failures or slowdowns. For example, if all components of a system temporarily slow down at the same time, it might not trigger alerts because the overall latency remains within acceptable bounds.

To address these issues, we need to instrument distributed monitoring more effectively. Here’s how:

Segmentation and Granularity: Implement fine-grained monitoring that breaks down metrics by specific segments such as customer IDs or request types. This allows us to track performance not just on an aggregate level but also at the individual user or transaction level.

Latency Distribution Analysis: Instead of relying solely on average latency, we should analyze the full distribution of latencies using percentiles (e.g., 95th percentile). Percentiles provide a better picture of tail behavior and can highlight when a significant portion of requests are experiencing degraded performance.

Root Cause Analysis Tools: Utilize advanced monitoring tools that not only collect but also correlate data across different parts of the system. This helps in pinpointing where the degradation is occurring, whether it’s within a single service or due to interactions between services.

Decentralized Monitoring: Introduce decentralized monitoring frameworks like Prometheus or OpenTelemetry that can be deployed closer to the points of interest (e.g., at the edge of the network). This reduces latency and ensures more accurate data capture for critical segments.

Real-Time Alerts and Notifications: Set up real-time alerting systems triggered by specific thresholds in the distribution analysis. For example, if the 95th percentile latency exceeds a certain threshold, an alert should be generated immediately, even if the average remains healthy.

Here’s a simplified architecture diagram to illustrate this setup:

Application
Service A
Service B
Database
Distributed Monitoring

By implementing these strategies, we can ensure that any degradation in performance for specific segments of users or traffic is immediately detected and addressed, preventing issues like the one described from going unnoticed.

**Comparison:**

```
Time	Latency (ms)
2023-10-01T14:00	50
2023-10-01T14:05	60
2023-10-01T14:10	70
2023-10-01T14:15	80
```

**Code:**

```python
# Example code to calculate percentiles using Python's statistics module
import statistics

def calculate_percentile(latencies, percentile):
    return statistics.percentile(latencies, percentile)

# Sample usage
latencies = [10, 20, 30, 40, 50, 60, 70, 80, 90, 100]
print(f"95th Percentile: {calculate_percentile(latencies, 95)}")
```

**Example input / output:**

```text
Input:
{
  "customer_segment": {
    "id": "CUST-123",
    "requests": [
      {"timestamp": "2023-10-01T14:00:00Z", "latency_ms": 50},
      {"timestamp": "2023-10-01T14:05:00Z", "latency_ms": 60},
      {"timestamp": "2023-10-01T14:10:00Z", "latency_ms": 70},
      {"timestamp": "2023-10-01T14:15:00Z", "latency_ms": 80}
    ]
  }
}

Output:
95th Percentile Latency for Customer Segment CUST-123: 70 ms
```

**Data:**

```json
{
  "customer_segment": {
    "id": "CUST-123",
    "requests": [
      {"timestamp": "2023-10-01T14:00:00Z", "latency_ms": 50},
      {"timestamp": "2023-10-01T14:05:00Z", "latency_ms": 60},
      {"timestamp": "2023-10-01T14:10:00Z", "latency_ms": 70},
      {"timestamp": "2023-10-01T14:15:00Z", "latency_ms": 80}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Request
4. Service A
5. Service B
6. Database
7. Query Execution Time
8. Latency > Threshold?
9. Alert
10. Continue Monitoring

