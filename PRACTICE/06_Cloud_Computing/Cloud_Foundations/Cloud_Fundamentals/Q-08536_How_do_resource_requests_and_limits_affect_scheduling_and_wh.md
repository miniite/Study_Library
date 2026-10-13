---
type: "practice"
difficulty: "Deep"
---

# How do resource requests and limits affect scheduling and what happens when a pod exceeds its memory limit?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Autoscaling Limits](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Autoscaling_Limits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with cloud-based container orchestration, such as Kubernetes, resource requests and limits play critical roles in ensuring efficient resource utilization, preventing overcommitment, and maintaining cluster stability. Understanding how these settings affect scheduling decisions is essential for backend engineers, especially those at a principal level where strategic decisions about scaling and performance are crucial.

#### Triaging the Question

The question revolves around two key concepts: resource requests and limits, specifically in the context of Kubernetes pods. The interviewer is probing your understanding of how these parameters interact with the scheduler to make placement decisions. Additionally, there's an interest in what happens when a pod exceeds its memory limit, which ties into the concept of resource overcommitment.

#### First-Principles Solution
Resource Requests and Limits
Resource Requests: These are hints to the scheduler about minimum resource requirements for a container. They don't guarantee that the resources will be allocated; they just provide information that can influence scheduling decisions.
Resource Limits: These are hard caps on the amount of resources a pod or container is allowed to use. Exceeding these limits may result in termination, capping, or other interventions by Kubernetes.
Scheduling Decisions
When Scheduling Pods:
The scheduler considers both requests and limits when deciding where to place pods.
It aims to find nodes that can meet the request while not exceeding the limit. This is why understanding the resource demands of your applications is critical for efficient deployment and scaling.
Pod Overcommitment
Memory Limits Exceeded: When a pod exceeds its memory limit, Kubernetes will take action based on the policy configured in the PodDisruptionBudget (PDB) or other policies.
By default, pods exceeding their limits may be terminated by the kubelet to free up resources and prevent overcommitment of the node's resources.
This can lead to pod restarts, which might affect service availability.

#### Closing Summary

In summary, resource requests and limits are vital for managing containerized applications in Kubernetes. They help the scheduler make informed decisions about where pods should be scheduled while preventing overcommitment that could lead to instability or downtime. Proper configuration of these parameters is crucial for maintaining performance, reliability, and scalability in your cloud environments.

#### Common Mistakes/Follow-Ups
Overcommitting Resources: Failing to set appropriate limits can lead to nodes running out of resources, which can result in degraded service.
Pod Restart Policies: Understanding the restart policies for pods when they exceed their resource limits is important to minimize disruption to your applications.
Monitoring and Logging: Effective monitoring and logging are necessary to detect and address issues related to resource overcommitment proactively.
Triaging Follow-Ups

The interviewer might ask about specific Kubernetes configurations or tools that help manage these scenarios, such as cAdvisor for detailed metrics or Prometheus for alerting. They could also probe deeper into the trade-offs between setting high limits versus risking node instability.

**Comparison:**

```
Resource	Request	Limit
Memory	64 MiB	512 MiB
CPU	250m	1000m
Node Resources (Scenario)
Memory: 8GB
CPU: 4 cores
Pod Resources (Scenario)
Memory Request: 64 MiB
Memory Limit: 512 MiB
CPU Request: 250m
CPU Limit: 1000m
```

**Code:**

```python
# Example of a Kubernetes resource configuration snippet in YAML
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "1000m"
```

**Example input / output:**

```text
Input: A pod with the following resource configuration is scheduled on a node.

Example YAML:
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "1000m"

Node with available resources:
- Memory: 8GB
- CPU: 4 cores

Output: The pod will be scheduled on the node because its requests are met, and it won't exceed the node's capacity.

Scenario where a pod exceeds its limit:
Pod Configuration (Exceeding Limit):
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    resources:
      limits:
        memory: "8Gi"

Node with limited capacity:
- Memory: 4GB

Output: The pod will be terminated because it exceeds the node's memory limit.
```

**Data:**

```yaml
# Example of a pod resource configuration
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  containers:
  - name: example-container
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "512Mi"
        cpu: "1000m"
```

**Flow:**

1. Meets Requests and Limits
2. Exceeds Memory Limit
3. Termination
4. Capping
5. Node Scheduler
6. Pod Requests vs NodeCapacity
7. Schedule Pod
8. Termination or Capping
9. Terminate Pod, Free Resources
10. Cap Resource Usage, PreventOvercommitment

