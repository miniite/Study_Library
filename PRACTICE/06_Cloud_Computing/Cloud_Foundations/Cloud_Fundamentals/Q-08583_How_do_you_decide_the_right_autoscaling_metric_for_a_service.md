---
type: "practice"
difficulty: "Deep"
---

# How do you decide the right autoscaling metric for a service whose load is measured in queue depth rather than CPU?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Autoscaling Metrics](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To decide the right autoscaling metric for a service where the load is measured by queue depth, we need to understand that our primary goal is to maintain an appropriate level of service availability and performance while minimizing unnecessary costs. In such scenarios, the key metric should be focused on the queue length rather than CPU utilization or other typical metrics like requests per second.

Firstly, let's break down what we're dealing with: a service where tasks are processed from a queue, and the load is measured by how many items are in this queue. The right autoscaling metric would ensure that the queue remains below a certain threshold to prevent backlog and ensure timely processing of tasks.

#### Triage

The question is essentially asking for the most appropriate autoscaling strategy when dealing with a workload characterized by an increasing queue depth. The interviewer is probing our understanding of how different metrics interact, particularly in environments where task queues are used extensively (e.g., background job processing systems).

#### Assumptions
and Context
Queue Depth: The primary metric to monitor.
Service Availability: Ensuring tasks are processed in a timely manner.
Cost Efficiency: Minimizing resource usage while maintaining service quality.
Step-by-Step Solution from First Principles

Define the Service-Level Objectives (SLOs):

Determine what "timely" means for your application. For example, if an SLO is to process 95% of tasks within a minute, you need to ensure that queue depth does not exceed a level where this objective is at risk.

Choose the Autoscaling Metric:

Since the workload is measured by queue depth, the autoscaler should target keeping the queue length below an acceptable threshold.
This can be achieved using cloud-native services like AWS Lambda with API Gateway or Kubernetes with custom metrics and horizontal pod autoscalers (HPA).

Implement a Custom Autoscaler:

Use a custom metric provider to feed the queue depth into your autoscaling service. For instance, in Kubernetes, you could use kubectl autoscale with a custom metric like queue-length.
Define a HPA that scales based on the queue length:
YAML
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: job-processor-autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: job-processor
  minReplicas: 1
  maxReplicas: 50
  metrics:
  - type: Pods
    pods:
      metricName: queue-length
      targetAverageValue: 5 # Adjust based on your SLOs

Monitor and Tune the Autoscaler:

Continuously monitor the queue depth to ensure that it remains within acceptable bounds.
Use Prometheus or another monitoring tool to visualize the queue length over time, helping you tune the autoscaling thresholds as needed.

#### Cons
ider Edge Cases:

Peak Loads: During peak times, make sure your autoscaler can handle sudden increases in workload without causing too many pods to be spun up.
Cold Starts: If cold start times are high for your service (e.g., AWS Lambda functions), ensure that the queue depth does not build up excessively before new instances can process tasks.

#### Closing Summary

In conclusion, deciding the right autoscaling metric for a service based on queue depth involves defining clear SLOs and implementing an autoscaler that targets these metrics. This approach ensures that your service remains responsive while optimizing resource usage.

#### Common Mistakes/Follow-Ups
Ignoring Queue Depth: Focusing solely on CPU or other traditional metrics without considering the underlying workload.
Over-Autoscaling: Spinning up too many instances, which can lead to wasted resources and increased costs.
Under-Autoscaling: Not scaling enough to handle peak loads, leading to backlog and poor service performance.

**Comparison:**

```
Metric Name	Description
queue-length	The number of tasks in the processing queue.
```

**Code:**

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: job-processor-autoscaler
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: job-processor
  minReplicas: 1
  maxReplicas: 50
  metrics:
  - type: Pods
    pods:
      metricName: queue-length
      targetAverageValue: 5 # Adjust based on your SLOs
```

**Example input / output:**

| Queue Length | Decision |
|--------------|----------|
| <5           | No action (green) |
| >=10         | Add one instance (yellow) |
| >=20         | Add two instances (red) |

**Data:**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: job-processor-service
spec:
  selector:
    app: job-processor
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
```

**Flow:**

1. No
2. Yes
3. Start
4. Is Queue Length > Threshold?
5. No Action (Green)
6. Add Instances
7. Monitor Queue Length
8. End

