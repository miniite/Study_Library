---
type: "practice"
difficulty: "Deep"
---

# How would you implement a health-check design that distinguishes liveness from readiness so Kubernetes doesn't kill a pod that's merely waiting on a slow dependency at startup?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ HttpClient and Resilient Outbound Calls](<../../../../TOOLS/DotNet/DotNet_Platform/HttpClient_and_Resilient_Outbound_Calls.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To effectively distinguish between liveness and readiness in a .NET application, we'll design a health-check mechanism using Kubernetes' liveness and readiness probes. The liveness probe ensures the container is running, while the readiness probe checks if it's ready to serve requests. By configuring these probes correctly, we can avoid unnecessary restarts for pods that are still initializing.

#### Triaging the Question

The core of this question revolves around understanding Kubernetes' health-check mechanisms and how they interact with a .NET application during startup. The interviewer is probing your knowledge on:

How liveness and readiness probes work.
Best practices for configuring these probes in a production environment.
Potential pitfalls and common mistakes when implementing health checks.

#### First Principles Solution

Understanding Liveness vs Readiness:

Liveness Probe: Checks if the container is running. Kubernetes will restart a pod if this probe fails, indicating that the application has crashed or isn't responding.
Readiness Probe: Ensures the application is ready to handle traffic. If this probe fails, Kubernetes routes traffic away from the pod until it passes.

Configuring Liveness and Readiness Probes:

For a .NET application starting up slowly due to external dependencies, we should configure liveness and readiness probes appropriately.
The livenessProbe can be configured with a shorter interval (e.g., 10-30 seconds) to ensure the pod is not terminated if it's just waiting for initialization.
The readinessProbe should have a longer interval (e.g., 60-120 seconds) and more complex checks that only pass once the application is fully initialized.

Health Check Endpoints:

A simple HTTP endpoint can be created in your .NET application to handle health-check probes.
The livenessProbe might just check if the container is running (/healthz/liveness).
The readinessProbe would perform more comprehensive checks, such as database connectivity or service availability (/healthz/readiness).

#### Example
Configuration:

YAML
apiVersion: v1
kind: Pod
metadata:
  name: my-net-app
spec:
  containers:
    - name: netapp-container
      image: my-net-app-image
      livenessProbe:
        httpGet:
          path: /healthz/liveness
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 30
      readinessProbe:
        httpGet:
          path: /healthz/readiness
          port: 8080
        initialDelaySeconds: 60
        periodSeconds: 60

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, by carefully configuring liveness and readiness probes in Kubernetes for a .NET application, we can ensure that slow-starting pods are not unnecessarily terminated. This approach balances the need to keep failed containers running with the requirement to route traffic only when the application is fully ready.

#### Common mistakes
include:

Configuring both probes too aggressively, leading to false positives and unnecessary restarts.
Not having a proper health-check endpoint in place, resulting in incorrect probe statuses.
Ignoring edge cases where external dependencies might delay readiness but not liveness.

#### Follow-ups
could include discussing how to handle retries, backoff strategies, or integrating custom health checks with application-specific logic.

**Code:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-net-app
spec:
  containers:
    - name: netapp-container
      image: my-net-app-image
      livenessProbe:
        httpGet:
          path: /healthz/liveness
          port: 8080
        initialDelaySeconds: 10
        periodSeconds: 30
      readinessProbe:
        httpGet:
          path: /healthz/readiness
          port: 8080
        initialDelaySeconds: 60
        periodSeconds: 60
```

**Example input / output:**

```text
livenessProbe:
Path: /healthz/liveness
Port: 8080
Initial Delay: 10 seconds
Period Interval: 30 seconds

readinessProbe:
Path: /healthz/readiness
Port: 8080
Initial Delay: 60 seconds
Period Interval: 60 seconds
```

