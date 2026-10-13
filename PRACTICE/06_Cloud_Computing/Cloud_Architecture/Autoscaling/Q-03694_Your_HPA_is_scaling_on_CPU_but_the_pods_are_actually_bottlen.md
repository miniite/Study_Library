---
type: "practice"
difficulty: "Deep"
---

# Your HPA is scaling on CPU but the pods are actually bottlenecked on a downstream database connection pool. Walk me through how you'd redesign the scaling signal.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Choosing Autoscaling Signals](<../../../../06_Cloud_Computing/Cloud_Architecture/Autoscaling/Choosing_Autoscaling_Signals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where horizontal pod autoscaling (HPA) based on CPU is not effective due to database connection pool bottlenecks, we need to rethink our approach to scaling signals. The key idea here is to design a more comprehensive and context-aware scaling signal that can better reflect the true load conditions of the application.

Firstly, let's clarify what HPA and its limitations are:

HPA scales the number of pods based on CPU utilization, which is straightforward but may not capture other critical performance metrics.
Database connection pool bottlenecks indicate a limit in concurrent database interactions, suggesting that we need to monitor and scale based on different metrics.
Triaging the Problem

The core issue here is that HPA only considers CPU utilization, which does not account for downstream dependencies like database connections. We need to redesign our scaling signal to include more relevant metrics such as connection pool usage, request latency, or queue lengths.

#### Assumptions
and Key Metrics:
Downstream Database: The application is making a significant number of requests to a database.
Connection Pool Limitation: There is a defined limit on the number of concurrent connections that can be made to the database.
Performance Metrics: We need metrics like connection pool usage, request latency, and queue lengths.

#### First-Principles Solution

Collect Detailed Metrics:

Use Prometheus or similar monitoring tools to collect detailed metrics such as the number of active connections in the database pool, request latency, and queue lengths.

Custom Scaling Signal:

Create a custom horizontal pod autoscaler (HPA) based on these new metrics. This can be achieved using Kubernetes Custom Metrics API or by implementing a custom HPA logic.

Dynamic Configuration:

The configuration for the custom scaling signal should dynamically adjust based on observed metrics, ensuring that we scale the number of pods to meet the current demand without hitting the database connection limits.

Cost and Performance Trade-Offs:

Balancing cost (more pods mean higher cost) with performance is crucial. Ensure that you only scale when necessary and use strategies like canary deployments or gradual scaling to avoid abrupt changes.

#### Implementation
Steps

Define Metrics for Scaling:

YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: custom-scaler
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Pods
    pods:
      metricName: database-connections
      targetAverageValue: 80

Implement Custom Logic:

Write a custom scaling logic that can be triggered by Kubernetes Jobs or CronJobs to periodically check the connection pool usage and dynamically adjust the HPA settings.

Monitor and Optimize:

Continuously monitor the performance and make adjustments as needed based on real-world usage patterns.

#### Closing Summary

In summary, we need to move beyond simple CPU-based scaling by implementing a custom scaling signal that factors in metrics like database connection pool utilization, request latency, and queue lengths. This approach ensures that our application can scale appropriately without running into bottlenecks caused by limited database connections.

#### Common Mistakes/Follow-Ups
Over-Scaling: Scaling too aggressively based on incorrect or incomplete metrics.
Under-Scaling: Failing to scale when necessary due to incorrect threshold settings.
Complexity in Implementation: Custom scaling logic can introduce complexity and potential performance overhead if not properly designed.

