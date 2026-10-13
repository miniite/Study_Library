---
type: "practice"
difficulty: "Deep"
---

# Explain how a Kubernetes HPA scaling on CPU can oscillate when the workload is actually latency-bound behind a slow dependency. How would you pick a better scaling signal and add stabilization?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Partition Lag and Hot Keys](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Partition_Lag_and_Hot_Keys/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of Kubernetes Horizontal Pod Autoscaler (HPA) oscillating due to a latency-bound workload, we need to understand that HPA primarily scales based on CPU utilization metrics, which may not always reflect the true performance bottlenecks in distributed systems. In this scenario, where the system is constrained by slow dependencies rather than CPU, traditional HPA can lead to frequent scaling adjustments and even oscillations.

#### Triaging the Question

The question asks us to identify a better scaling signal and suggest ways to stabilize the system. We need to consider that:

HPA's Metrics: The current metric used is CPU utilization, which doesn't capture latency issues.
Latency Constraints: The workload is actually bottlenecked by external dependencies, leading to unpredictable delays rather than CPU exhaustion.
Oscillations: Frequent scaling adjustments can cause the system to overshoot and undershoot its optimal resource allocation.

#### First-Principles Solution

To address this problem, we need to:

Identify a Suitable Scaling Signal: A more relevant metric like latency or response time could be used.
Add Stabilization Mechanisms: Implement strategies such as hysteresis, time-based smoothing, and rate limiting.
1. Identifying a Better Scaling Signal
Latency Metrics: Use metrics that directly correlate with the system's performance, such as response time or latency.
External Dependency Metrics: Collect statistics from external systems to understand their impact on the workload.
2. Implementing Stabilization Mechanisms
Hysteresis: Introduce a threshold for scaling actions so that small fluctuations in the metrics do not trigger frequent adjustments.
Time-Based Smoothing: Average the scaling signal over time to reduce noise and false positives.
Rate Limiting: Cap the rate at which resources can be added or removed to avoid extreme changes.
Detailed Steps

Collect Metrics:

Collect CPU, memory, and network metrics for baseline monitoring.
Monitor latency, response time, and any external dependency metrics (e.g., slow database queries).

Set Up Custom Monitoring:

Configure Prometheus or similar monitoring tools to collect the required metrics.
Use custom alerts or dashboards to visualize these metrics in real-time.

Implement HPA with Alternative Metrics:

Update the Kubernetes HPA configuration to use a more relevant metric, such as latency.
YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metricName: latency
      targetAverageValue: 5ms # Example threshold

Add Stabilization Mechanisms:

Implement hysteresis by adding a targetHysteresis field to the HPA configuration.
YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metricName: latency
      targetAverageValue: 5ms
      targetHysteresis: 2ms # Hysteresis to avoid frequent adjustments
Use time-based smoothing by averaging the metric over a window period.
YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metricName: latency
      targetAverageValue: 5ms
      targetHysteresis: 2ms
      averageWindow: 60s # Smoothing over a 60-second window

#### Closing Summary
and Common Mistakes/Follow-Ups

By implementing these strategies, we can better align the scaling actions with the actual performance bottlenecks in our system. Common mistakes include relying solely on CPU metrics, neglecting external dependency metrics, and not adding sufficient stabilization mechanisms to prevent oscillations.

To further discuss this solution:

Performance Testing: Conduct A/B testing using different scaling signals and stabilization parameters.
Real-time Monitoring: Continuously monitor the system's performance to adjust the thresholds as needed.
Cost Optimization: Evaluate the impact of these changes on resource utilization and cost.

**Comparison:**

```
Time	Latency (ms)
t0	10
t1	8
t2	7
t3	9
t4	11
t5	6
```

**Code:**

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Pods
    pods:
      metricName: latency
      targetAverageValue: 5ms
      targetHysteresis: 2ms
      averageWindow: 60s
```

**Example input / output:**

```
Input Metrics	Target Average Value	Hysteresis	Average Window
[10, 8, 7, 9, 11, 6]	5ms	2ms	60s
Response
PLAINTEXT
Copy
Based on the provided metrics:
- The average latency over the last minute is approximately 9ms.
- This value is above the targetAverageValue of 5ms, but within the hysteresis threshold (2ms).
- No scaling action is triggered due to stabilization.

The system remains stable without frequent adjustments.
```

**Data:**

```json
{
  "latency": [10, 8, 7, 9, 11, 6],
  "targetAverageValue": 5,
  "targetHysteresis": 2,
  "averageWindow": 60
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Monitor Latency
6. Is Latency Above Target?
7. Apply Scaling Action
8. Smooth Latency Over Window
9. Is Smoothed Latency AboveThreshold + Hysteresis?
10. Do Not Scale

