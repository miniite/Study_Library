---
type: "practice"
difficulty: "Deep"
---

# Your gateway autoscales on CPU but a spike of TLS handshakes saturates it before CPU crosses the HPA threshold; what custom metric would you scale on and how do you avoid flapping?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Scaling for Traffic Spikes](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/Gateway_Scaling_for_Traffic_Spikes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary:

To address the issue where a surge in TLS handshakes causes saturation before CPU usage triggers autoscaling, we need to identify an appropriate custom metric that more accurately reflects the actual load on the gateway. This involves understanding the different types of load and identifying metrics that can better correlate with service health.

#### Triaging the Question

The question is probing your ability to understand the nuances of cloud-native application scaling and how to fine-tune autoscaling policies beyond default CPU-based metrics. The interviewer wants to see if you can identify a more relevant metric, such as TLS handshake rate or connection count, that can prevent "flapping" (frequent and rapid scaling actions) caused by transient load spikes.

#### Assumptions
Autoscaler is configured based on CPU utilization.
Gateway experiences high traffic with frequent TLS handshakes.
The goal is to improve stability and reduce unnecessary scaling events.
Step-by-Step Solution:
1. Understand the Problem

TLS handshakes are a critical part of HTTPS traffic but do not directly translate into CPU usage. When there's a spike in TLS handshakes, it can quickly overload the gateway without necessarily causing significant CPU load.

2. Identify Relevant Metrics
TLS Handshake Rate: Measures how many new connections or handshakes are initiated per second.
Connection Count: Tracks the total number of active connections to the gateway.
Latency: Measures the response time of requests, which can indicate whether there's enough capacity.
3. Implement Custom Metrics
Set Up Prometheus or another monitoring tool to collect these metrics.
Define a custom scaling policy using Kubernetes Horizontal Pod Autoscaler (HPA) with an additional metric configuration:
YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: gateway-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: gateway-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: External
    external:
      metric:
        name: tls_handshake_rate
      targetAverageValue: 500/s
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
4. Validate the Solution
Dry Run: Simulate a TLS handshake spike and observe if the gateway scales appropriately.
Monitor Performance: Use Grafana or similar dashboard to monitor the new metrics over time.
5. Refine the Scaling Policy

Based on real-world data, refine the scaling thresholds:

Adjust the targetAverageValue for TLS handshakes based on observed performance.
Set a minimum delay between scale-up and scale-down actions to avoid flapping.

#### Closing Summary

By introducing custom metrics like TLS handshake rate or connection count, you can more accurately scale your API gateway, ensuring that it handles transient load spikes without unnecessary scaling events. This approach improves both the stability of the service and reduces the risk of resource wastage from frequent autoscaling actions.

#### Common Mistakes/Follow-Ups
Forgetting to validate the solution in a controlled environment before applying it broadly.
Setting overly aggressive thresholds that lead to constant flapping or under-provisioning.
Not considering other metrics like latency which can provide additional context for scaling decisions.

**Code:**

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: gateway-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: gateway-deployment
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: External
    external:
      metric:
        name: tls_handshake_rate
      targetAverageValue: 500/s
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 50
```

