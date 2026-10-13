---
type: "practice"
difficulty: "Expert"
---

# A deploy that changes connection pool sizing exhausts the database's max connections when old and new pods overlap during rollout; how do you account for transient double-capacity in your deploy plan?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Zero_Downtime_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

When rolling out a Kubernetes deployment that changes connection pool sizing, it’s crucial to account for the temporary state where both old and new pods exist simultaneously. This overlap can exhaust the database's maximum connections if not managed properly. The solution involves carefully planning the rollout sequence, leveraging Kubernetes features like rollingUpdate, and potentially implementing custom logic in the application or deployment strategy.

#### Triaging the Question

The question is essentially about managing transient states during a Kubernetes deployment where old and new pods coexist for a period of time. This overlap can lead to temporary double-capacity usage on the database, which needs to be accounted for to avoid connection pool exhaustion errors. The interviewer is probing your understanding of Kubernetes deployment strategies, connection pooling, and how to manage resource constraints during rollouts.

#### Assumptions
We are using a Kubernetes environment with a rolling update strategy.
The application has a connection pool that dynamically adjusts based on pod state changes.
The database has a maximum number of allowed concurrent connections (max_connections).
The deployment involves changing the connection pool size from one version to another.

#### First-Principles Solution

Understand the Rollout Strategy: In Kubernetes, rolling updates are managed via rollingUpdate strategy in the deployment configuration. This ensures that pods are updated in a controlled manner, allowing new versions of the application to run alongside old ones until all pods have been replaced.

Calculate Transient Double-Capacity Period:

Determine the duration during which both old and new pods will coexist.
Calculate the maximum number of connections each pod type can use (old and new).
Ensure that the sum of these does not exceed the database's max_connections.

Implement Connection Pool Adjustments:

In your application code, implement logic to dynamically adjust connection pool sizes based on whether a pod is old or new.
Use environment variables or config maps to provide different values for the old and new versions.

Configure Kubernetes RollingUpdate Settings:

Set the maxUnavailable and maxSurge parameters in your deployment manifest to control how many pods can be unavailable during the update.
YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  template:
    ...

Database Connection Monitoring:

Monitor the database's connection usage during the rollout to ensure it stays within limits.
Implement a monitoring alert that triggers if the number of active connections exceeds the threshold.

Graceful Pod Termination and Initialization:

Ensure pods are terminated gracefully, allowing them to release resources such as database connections before being replaced.
Use terminationGracePeriodSeconds in your pod specification to control how long a pod runs after receiving the termination signal.

Testing and Validation:

Perform thorough testing of the rollout process using tools like Istio or Kubernetes Ingress to simulate different scenarios and validate that the connection pool adjustments work as expected.
Use load tests to simulate high traffic conditions during the rollout to ensure stability.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

#### Common Mistakes

Failing to account for the transient state where old and new pods coexist, leading to temporary double-capacity issues.
Not configuring maxUnavailable and maxSurge correctly in the deployment manifest, resulting in too many pods being unavailable simultaneously.
Overlooking graceful termination logic, causing unnecessary resource contention.

#### Follow-Ups

How would you handle more complex scenarios with multiple services or databases?
What other Kubernetes features could be leveraged to further optimize this process?
Discuss strategies for handling different types of applications (e.g., stateless vs. stateful).

**Comparison:**

```
Phase	Old Pods	New Pods	Max Connections
Before Update	8	0	8
During Rollout	7	1	9
During Rollout	6	2	10
During Rollout	5	3	11
...	4	4	16
Final Phase	1	4	17
```

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 25%
      maxSurge: 25%
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app-container
        image: my-app-image:v1
        env:
          - name: CONNECTION_POOL_SIZE
            value: "10"
```

**Example input / output:**

```text
Input:
- Current deployment with `old_pool_size` = 8.
- New deployment with `new_pool_size` = 12.

Output:
- During the rollout, both old and new pods coexist for a period where the total connection usage can temporarily reach up to 20 connections (8 + 12).
- Ensure database `max_connections` is set to at least 20 to accommodate this overlap safely.
```

**Data:**

```yaml
max_connections: 20
old_pool_size: 8
new_pool_size: 12
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Deployment Initiation
6. Are old and new podscoexisting?
7. Monitor connection usage
8. Continue deployment withoutconcerns
9. Check if max_connections isexceeded
10. Alert on high connectionusage

