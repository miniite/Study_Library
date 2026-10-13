---
type: "practice"
difficulty: "Deep"
---

# Your rollout uses a Kubernetes Deployment with maxSurge and maxUnavailable defaults and it briefly halves capacity at peak traffic; how would you tune the rollout so it never dips below what live traffic needs?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Zero_Downtime_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary TL;DR:

In this scenario, we need to ensure that our Kubernetes Deployment rollout maintains sufficient capacity during a brief half-capacity dip at peak traffic times. We will leverage maxSurge and maxUnavailable settings effectively while possibly adjusting other deployment strategies like rolling updates and pod readiness checks.

#### Triaging the Question

The core of this question revolves around ensuring high availability and minimizing downtime during rollouts in Kubernetes. The key is to balance between the risk of not having enough live capacity versus potentially losing a significant portion of our service's traffic momentarily. We will need to fine-tune maxSurge and maxUnavailable, understand their implications, and consider additional strategies like adjusting the rollout rate.

#### Assumptions
Application Architecture: A typical microservices architecture with multiple replicas.
Traffic Patterns: Peak traffic is sudden and brief (e.g., due to a surge in user activity during peak business hours).
Kubernetes Resources: Using Deployments, ReplicaSets, or StatefulSets if stateful.
Step-by-Step Solution from First Principles:

Understanding maxSurge and maxUnavailable:

maxSurge: Allows a certain percentage of additional replicas to be created during the rollout.
maxUnavailable: Ensures that at any given time, no more than this percentage of the total number of available pods is unavailable.

Current Setup:

With maxSurge and maxUnavailable defaults, Kubernetes will allow a certain number of new replicas to be created without fully rolling out all updates immediately.
The brief half-capacity dip indicates that during peak traffic, the application might temporarily handle less than 50% of its live traffic.

Adjusting maxSurge and maxUnavailable:

Increase maxSurge to ensure there are enough new replicas being created quickly.
Decrease maxUnavailable to minimize downtime by ensuring more replicas remain available during the rollout.

Additional Strategies:

Rollout Rate Limit: Gradually increase the number of pods that are updated to avoid overwhelming the system with too many updates at once.
Pod Readiness Probes: Ensure only healthy, ready pods handle traffic until all pods have been successfully updated and marked as healthy.

#### Example
Configuration:

YAML
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 10%
      maxUnavailable: 30%
  template:
    spec:
      containers:
        - name: my-app
          readinessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10

Monitoring and Feedback Loops:

Continuously monitor the health of the application during rollouts.
Use metrics like kubectl top pods to ensure resource utilization is within acceptable limits.

#### Closing Summary
+ Common Mistakes/Follow-Ups:

To prevent capacity dips, we tuned maxSurge and maxUnavailable, and considered rollout rate limits. Properly configuring these settings ensures that the application remains responsive during rollouts. However, it's crucial to validate these configurations in a staging environment before applying them to production.

#### Common mistakes
include:

Overlooking the need for maxSurge and maxUnavailable adjustments.
Failing to implement effective pod readiness probes.
Not monitoring resource usage post-rollout.
Triaging Follow-Ups:
What metrics would you use to validate this setup?
How would you handle more severe traffic surges (e.g., 3x normal)?
Are there any additional strategies besides maxSurge and maxUnavailable that could be considered?

**Comparison:**

| Pod Readiness Probes | |----------------------|-----------| | Path: /healthz | Port: 8080 | | Initial Delay: 5s | Period: 10s|

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
      maxSurge: 25%
      maxUnavailable: 30%
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app-image:v1
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
```

**Example input / output:**

```text
# Example dry-run input and output

Input:
- Deploy a new version of the application with maxSurge set to 25% and maxUnavailable set to 30%.

Output:
- During deployment, Kubernetes creates additional replicas (up to 1.25 * current replicas) while ensuring no more than 30% of total capacity is unavailable.
- This configuration minimizes the risk of dropping below necessary live traffic during peak times.
```

**Data:**

```yaml
# Example deployment configuration with adjusted maxSurge and maxUnavailable
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-deployment
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: "25%" # Allow for 25% more pods than the current number
      maxUnavailable: "30%" # Ensure no more than 30% of total capacity is unavailable
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-app
        image: my-app-image:v1
        readinessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
```

**Flow:**

1. Deploy New Version
2. maxSurge: 25%
3. maxUnavailable: 30%
4. Create Additional Replicas
5. Maintain at Least 70% LiveCapacity
6. Monitor Health with ReadinessProbes

