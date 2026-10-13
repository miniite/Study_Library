---
type: "practice"
difficulty: "Intermediate"
---

# Your gateway's health check to Kubernetes passes but requests fail because the readiness probe checks the wrong port; how do you design gateway liveness vs readiness probes so a bad probe never blackholes traffic silently?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Connection Reuse and Client Errors](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Connection_Reuse_and_Client_Errors/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue where the gateway's health check to Kubernetes passes but requests fail due to incorrect readiness probe settings, we need to carefully design both liveness and readiness probes. The key is ensuring that these probes accurately reflect the application’s state so as not to inadvertently blackhole traffic.

#### Triage

The question is asking how to properly configure liveness and readiness probes in an API Gateway deployed on Kubernetes. The primary concern here is making sure the liveness probe does not fail due to a misconfigured readiness probe, which could lead to traffic being blackholed silently if the application starts up but fails at runtime.

#### Assumptions
You are using an Nginx-based or similar API Gateway.
The application running behind the gateway exposes both a health check endpoint and runs on a specific port (e.g., 8080).
The current readiness probe is configured to use a different port than where the application actually runs.

#### First-Principles Solution

Understand the Purpose of Probes:

Liveness Probe: Checks if the container is running.
Readiness Probe: Ensures the service is ready to receive traffic.

Correct Setup for Readiness Probe:

The readiness probe should be configured with a path that checks the application's health (e.g., /healthz).
Ensure it points to the correct port where the application runs.

Liveness and Readiness Probes Configuration:

YAML
apiVersion: v1
kind: Pod
metadata:
  name: my-api-gateway
spec:
  containers:
  - name: api-gateway
    image: nginx
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      failureThreshold: 3
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
      failureThreshold: 3

Handling Edge Cases:

False Positives: Ensure the liveness probe does not falsely declare the container as unresponsive.
False Negatives: Avoid the readiness probe incorrectly marking the service as ready before it is fully operational.

Monitoring and Logging:

Implement robust monitoring for both probes to catch any issues early.
Use logging to track when probes fail or succeed, aiding in diagnosing problems.

#### Closing Summary

Properly configuring liveness and readiness probes ensures that your application remains available without blackholing traffic. By aligning the probes with the actual health-check path of your application and ensuring they are set up correctly, you can prevent unexpected outages due to misconfigurations.

#### Common Mistakes/Follow-Ups
Mistake: Misconfiguring the port for the readiness probe.

#### Follow-Up
Always verify that the configured ports match where your application runs. Also, ensure that the health check paths are correctly set up and tested before deploying.

**Code:**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-api-gateway
spec:
  containers:
  - name: api-gateway
    image: nginx
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
        initialDelaySeconds: 5
        periodSeconds: 10
      failureThreshold: 3
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
        initialDelaySeconds: 15
        periodSeconds: 20
      failureThreshold: 3
```

**Example input / output:**

```text
Current Setup:
- Readiness Probe: /healthz on port 9000 (application runs on 8080)
- Liveness Probe: /healthz on port 8080

Desired Setup:
- Readiness Probe: /healthz on port 8080
- Liveness Probe: /healthz on port 8080
```

