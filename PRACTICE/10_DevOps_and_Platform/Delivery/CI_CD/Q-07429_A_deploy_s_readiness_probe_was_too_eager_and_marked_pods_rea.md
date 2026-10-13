---
type: "practice"
difficulty: "Deep"
---

# A deploy's readiness probe was too eager and marked pods ready before caches warmed, spiking p99 for the first minute; how would you sequence cache warm-up before a pod takes traffic?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Zero Downtime Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Idempotent_Deploy_Jobs/Zero_Downtime_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a deploy's readiness probe marking pods as ready prematurely and causing spikes in the 99th percentile (p99) latency, we need to sequence cache warm-up before a pod starts taking production traffic. This involves carefully managing the initial deployment phase to ensure that caches are fully warmed up, thereby avoiding performance issues.

#### Triage
Understanding the Problem

The question is essentially asking how to properly manage the deployment process so that pods are only considered ready once their application caches are fully initialized. The core issue here is timing—getting the readiness probe right and ensuring that it doesn't declare a pod ready too soon, leading to performance degradation.

#### Assumptions
We have a microservices architecture with various services that rely on caches.
The application uses a health check mechanism (readiness probe) to determine when a pod is ready for traffic.
Cache warm-up time varies depending on the service and data being cached.
The current readiness probe configuration might be too aggressive, leading to premature marking of pods as ready.

#### First-principles Solution

To sequence cache warm-up before a pod starts taking traffic, we need to implement a delayed readiness probe with stages. This approach ensures that each component is fully initialized and ready for use before it handles production traffic. Here's a step-by-step breakdown:

Initial Deployment: Deploy the new version of the application using rolling updates.
Stage 1: Health Check Initialization:
Set up a readiness probe with an initial delay to allow the pod to initialize.
Stage 2: Cache Warm-Up:
Use a custom health check script or a service that periodically sends requests to the new application until its cache is fully initialized.
Stage 3: Gradual Traffic Shift:
Once the cache is warm, gradually shift traffic from old pods to new ones.

#### Implementation
Steps
Update Deployment YAML:
YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0%
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-app:latest
        readinessProbe:
          httpGet:
            path: /health-check
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
Custom Health Check Script:
Write a script that waits for the cache to be fully initialized before marking the pod as ready.
BASH
#!/bin/bash
until curl --silent http://localhost:8080/health-check-cache-status | grep "cache-warmed"; do sleep 5; done
echo "Cache is now warm."
Gradual Traffic Shift:
Use a rolling update strategy with a maxSurge and maxUnavailable to control the traffic shift.
YAML
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0%

#### Closing Summary
+ Common Mistakes/Follow-ups

By implementing a staged deployment process with delayed readiness probes and cache warm-up, we can ensure that pods are fully ready before they start handling production traffic. This approach helps in maintaining consistent performance and avoiding the initial spike in p99 latency.

#### Common mistakes
include:

Not allowing enough time for the cache to warm up.
Setting too aggressive health check intervals.
Failing to gradually shift traffic, leading to sudden spikes in load.

#### Follow-ups
could involve optimizing the health check script further or exploring more advanced deployment strategies like canary releases.

**Comparison:**

```
Stage	Time Delay	Action
1	60 seconds	Wait for pod initialization
2	N/A	Warm up cache using health check script
3	Gradual	Shift traffic to new pods after cache is warm
```

**Code:**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 0%
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-app:latest
        readinessProbe:
          httpGet:
            path: /health-check
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 10
```

**Example input / output:**

```text
Initial Deployment:
Deployment of my-app-74f5b8bc4d-6qk9l with initial delay set to 60 seconds.
Health Check Script Output:
Cache is now warm.

Deployment Stages:
1. Cache Initialization (60 seconds)
2. Health Check Confirmation: "Cache is now warm."
3. Traffic Gradual Shift
```

**Data:**

```json
{
  "podStatus": {
    "podName": "my-app-74f5b8bc4d-6qk9l",
    "status": "Initializing Cache"
  },
  "healthCheckResult": {
    "path": "/health-check-cache-status",
    "response": "Cache is now warm."
  }
}
```

**Flow:**

1. Initial Deployment
2. Wait 60 seconds
3. Warm up cache using customscript
4. Health check passes
5. Shift 25% traffic to new pods
6. Complete deployment

