---
type: "practice"
difficulty: "Deep"
---

# Explain what happens to in-flight requests when Kubernetes drains a node during a cluster autoscaler scale-down and how PodDisruptionBudgets preReadiness and connection draining together prevent dropped traffic.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Metastable Failures](<../../../../07_Distributed_Systems_and_Design/Distributed_Systems/Distributed_Systems_Core/Retry_Budgets_and_Backoff/Metastable_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand what happens to in-flight requests when Kubernetes drains a node during a cluster autoscaler scale-down, we need to delve into several key concepts, including how nodes are drained, the role of PodDisruptionBudgets (PDBs), and how preReadiness checks and connection draining work together to prevent dropped traffic. Let's break this down step by step.

Firstly, when Kubernetes performs a cluster autoscaler scale-down, it begins the process of draining a node. This involves gracefully shutting down pods on that node without causing disruption to ongoing application operations. The key steps here are:

Identifying Pods to Evict: The autoscaler identifies which pods can be safely evicted based on resource requirements and constraints.
Draining the Node: Once identified, a pod is marked for eviction, and its associated resources (like network connections) begin to be terminated.
Graceful Shutdown: Before fully terminating a pod, Kubernetes waits for preReadiness checks to confirm that it's safe to shut down the pod without causing disruptions.

PodDisruptionBudgets (PDBs) are used to ensure that certain applications can tolerate only a limited number of pod evictions at any given time. This is crucial in preventing sudden and significant disruptions when nodes are being drained, especially in critical services.

Now, let's dive into how preReadiness checks and connection draining work together:

PreReadiness Checks: These ensure that a pod can be safely terminated without causing application-level errors or data corruption. Typically, these checks involve waiting for the application to gracefully shut down before Kubernetes forces an eviction.

Connection Draining: This is a mechanism used by Kubernetes to gracefully terminate network connections from pods being evicted. It involves:

Monitoring Connections: Kubernetes monitors existing network connections and waits until they are closed naturally.
Quiescing New Requests: Any new requests are blocked or queued, ensuring no additional load is placed on the pod.

By combining these mechanisms, Kubernetes can ensure that in-flight requests to a pod being evicted are properly handled:

Existing connections from clients (e.g., web browsers, API clients) will be closed naturally.
No new connections are accepted until the old ones are fully terminated.
The application has time to handle any pending operations and gracefully shut down.

This process minimizes the risk of dropped traffic or service disruptions during a scale-down operation. However, it's essential to configure PDBs correctly to ensure that critical services can tolerate only a few pod evictions at once.

**Code:**

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: example-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: my-app
```

**Example input / output:**

```

A sample scenario:

Initial Setup: A Kubernetes cluster with two replicas of a web application running on nodes Node1 and Node2, each managed by a deployment.

Scale Down Triggered: The autoscaler triggers a scale-down operation due to resource utilization dropping below certain thresholds.

Node Draining: Node1 is marked for draining. The system starts the graceful shutdown process.

PLAINTEXT
Copy
[09:00] - PreReadiness checks initiated on Pod P1 running on Node1.
[09:05] - Last connection from client X to Pod P1 has been closed.
[09:10] - Kubernetes forces the termination of Pod P1, gracefully shutting down the application.

Result: During this period, no new requests are accepted by Node2; existing connections are allowed to close naturally. The service remains available, and users experience minimal disruption.
```

**Flow:**

1. Cluster Autoscaler TriggersScale Down
2. Identify Pods for Eviction
3. Node Draining Begins
4. PreReadiness Checks Initiated
5. Existing Connections Closed
6. Graceful Pod Termination
7. Service Continues withMinimal Downtime
8. Connection Draining Enabled

