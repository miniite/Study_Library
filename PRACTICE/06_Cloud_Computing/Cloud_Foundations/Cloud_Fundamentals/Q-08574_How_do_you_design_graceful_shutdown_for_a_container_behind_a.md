---
type: "practice"
difficulty: "Deep"
---

# How do you design graceful shutdown for a container behind a load balancer so in-flight requests finish and no new ones are dropped during a rolling deploy?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Control Plane Failures](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Safe_Config_Rollouts/Control_Plane_Failures.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a graceful shutdown for a container behind a load balancer, the primary goal is to ensure that all ongoing requests complete before the container gets removed from the service pool. This avoids premature termination of in-flight requests and ensures zero downtime during a rolling deployment.

This problem requires a deep understanding of cloud fundamentals, including load balancing strategies, container orchestration, and state management. Let's break down the solution step by step using first principles:

#### Triage

The question is asking how to handle the graceful shutdown of a container while managing in-flight requests. The interviewer is probing your knowledge of service reliability, load balancer configurations, and best practices for rolling updates.

#### Assumptions
We are dealing with a Kubernetes cluster running containers.
A load balancer (e.g., AWS ALB) is used to distribute incoming traffic.
Containers are part of a Deployment or StatefulSet in Kubernetes.
The application needs to handle both HTTP and gRPC requests.
The rolling update strategy involves gradually replacing old instances with new ones.

#### First Principles Solution
Service Discovery and Naming: Ensure your services are properly registered in a service mesh (like Istio) for advanced control over traffic routing.
Load Balancer Configuration: Configure the load balancer to allow health checks and configure it to drain unhealthy or terminating instances.
Rolling Update Strategy: Implement a rolling update strategy that gradually scales down old instances while scaling up new ones.
Grace Periods: Use Kubernetes' terminationGracePeriodSeconds to ensure in-flight requests have enough time to complete before the container is terminated.
Step-by-Step Solution

Health Checks:

Configure the load balancer with health checks that monitor the application's liveness and readiness.
Health checks should be configured to detect when an instance needs to be removed from the service pool.

Draining Instances:

When a rolling update is initiated, Kubernetes sends a SIGTERM signal to the old instances.
The terminationGracePeriodSeconds parameter in the pod specification controls how long the application has to complete its work before being terminated forcefully.
For example, if set to 30 seconds, the container must finish any ongoing requests within this period.

Rolling Update Strategy:

Gradually scale down the old instances while scaling up new ones.
This ensures that at least some replicas are always available for incoming traffic during the transition.
The rolling update can be configured to have a specific concurrency limit, ensuring that not all old pods are terminated simultaneously.

Configuration Example:

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
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-app:latest
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
      terminationGracePeriodSeconds: 30  # Graceful shutdown timeout

#### Closing Summary
and Common Mistakes/Follow-Ups
Grace Period: Ensure the grace period is adequate for your application’s needs. Too short, and requests may get dropped; too long, and it may delay the deployment.
Health Check Configuration: Proper health checks are crucial to detect when an instance should be terminated gracefully.
Rolling Update Concurrency: The concurrency setting affects how quickly the old instances can be replaced by new ones.

Common follow-ups might include:

How do you ensure stateless applications?
What if your application has stateful components like databases or caches?
How would you handle different types of requests (e.g., long-running tasks)?

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
      maxSurge: 1
      maxUnavailable: 0
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: my-container
        image: my-app:latest
        livenessProbe:
          httpGet:
            path: /healthz
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 5
          periodSeconds: 10
      terminationGracePeriodSeconds: 30  # Graceful shutdown timeout
```

**Flow:**

1. Yes
2. No
3. Initiate Rolling Update
4. Health Check Failure?
5. Graceful Shutdown (SIGTERM)
6. Wait for Grace Period
7. Scale Down Old Instances,Scale Up New Ones
8. Complete Rollout
9. Keep Instance Running

