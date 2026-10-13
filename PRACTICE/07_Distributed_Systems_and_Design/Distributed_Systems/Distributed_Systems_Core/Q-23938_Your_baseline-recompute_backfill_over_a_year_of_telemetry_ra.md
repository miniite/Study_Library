---
type: "practice"
difficulty: "Deep"
---

# Your baseline-recompute backfill over a year of telemetry ran on the same cluster as live detection and browned out the real-time path. Design an architecture that isolates and prioritizes the two so historical recompute doesn't impact time-to-detect.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Distributed Backfills](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Clock_Skew_and_Time_Windows/Distributed_Backfills.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this challenge, we need to design a resilient distributed system architecture that ensures historical backfill operations do not interfere with real-time detection. The core idea is to isolate these two processes and prioritize live detection while ensuring recompute can still be performed without causing downtime.

The question is essentially asking us to:

Isolate Processes: How can we separate the resources used by real-time detection from those used for historical backfill?
Prioritize Real-Time Detection: How can we ensure that real-time alerts and detections are always prioritized over slower, more resource-intensive recompute tasks?
Triaging the Problem
Current Setup: The baseline-recompute backfill is running on the same cluster as live detection.
Issue: The backfill operation has caused the real-time path to "brown out," indicating that it's overwhelming the system resources and disrupting real-time processing.

#### Assumptions

We have a finite number of compute nodes available in our cluster.
Real-time detection requires low-latency, high-throughput processing capabilities.
Historical backfill is CPU and I/O intensive but can tolerate higher latency.

#### First Principles Solution
Resource Allocation Strategy: Implement a dynamic resource allocation strategy that prioritizes real-time tasks over historical recompute tasks during peak times.
Cluster Partitioning: Physically or logically partition the cluster to isolate real-time processing from backfill operations.
Load Balancing and Scheduling: Use advanced load balancing techniques and scheduling policies to manage resources more effectively.
Step-by-Step Solution

Logical Isolation:

Use namespaces or virtual clusters to logically separate real-time and historical workloads.
Each namespace can have its own set of resource limits, ensuring that one workload does not dominate the others.

Dynamic Resource Allocation:

Implement a Kubernetes-based solution where pods for real-time detection are given higher priority in terms of CPU and memory allocation.
Use custom resource requests and limits to ensure that real-time tasks get preferential treatment.

Queue Management:

Introduce separate queues for historical backfill and real-time processing.
Prioritize the real-time queue over the historical queue, ensuring that any recompute task in progress is paused if a critical real-time task needs resources.

Monitoring and Alerting:

Set up detailed monitoring to track resource usage and alert when thresholds are breached.
Use automated scaling policies to adjust the number of instances based on current load.

#### Closing Summary

By implementing logical isolation, dynamic resource allocation, and prioritized queue management, we can ensure that historical backfill operations do not impact real-time detection. This approach leverages modern distributed systems practices such as Kubernetes for advanced resource management and prioritization.

#### Common Mistakes/Follow-Ups

Forgetting to implement monitoring and alerting can lead to undetected issues.
Not properly setting up priority queues can result in unnecessary resource contention between tasks.
Underestimating the impact of recompute on real-time processes can lead to suboptimal system performance.

**Comparison:**

```
Task Type	CPU Request (m)	Memory Request (Mi)
Real-Time Detection	500m	1Gi
Historical Backfill	200m	512Mi
```

**Code:**

```python
# Example code snippet for setting up custom resource requests and limits in Kubernetes
def set_resource_requests(pod_spec, cpu_request, memory_request):
    pod_spec.containers[0].resources.requests = {
        "cpu": f"{cpu_request}m",
        "memory": f"{memory_request}Mi"
    }

# Example usage in a deployment YAML file
apiVersion: apps/v1
kind: Deployment
metadata:
  name: real-time-detection-deployment
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: real-time-detection
    spec:
      containers:
      - name: detection-container
        image: my-realtime-detection-image
        resources:
          requests:
            cpu: "100m"
            memory: "256Mi"
```

**Example input / output:**

```text
Input: Real-time detection system receives 1GB of telemetry data every minute. Historical backfill job needs to recompute a year's worth of data, estimated at 30TB.

Output:
- Real-time processing maintains latency under 1 second for all detections.
- Historical backfill completes within expected time frame (without affecting real-time performance).
```

**Data:**

```yaml
# Example of a deployment manifest with priority and resource limits
apiVersion: batch/v1
kind: Job
metadata:
  name: historical-backfill-job
spec:
  template:
    metadata:
      labels:
        app: backfill
    spec:
      containers:
      - name: backfill-container
        image: my-historical-backfill-image
        resources:
          requests:
            cpu: "200m"
            memory: "512Mi"
```

**Flow:**

1. Real-time Detection
2. Priority Queue
3. Historical Backfill
4. Queue Management
5. Resource Allocation Strategy
6. Isolation Mechanisms

