---
type: "practice"
difficulty: "Deep"
---

# A GitOps deploy succeeded but users saw errors because readiness gates weren't wired, so Argo CD reported Healthy prematurely — how do you make health checks accurate?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Argo CD vs Flux](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To ensure that health checks are accurate in a GitOps deployment with Argo CD, we need to properly configure readiness gates and liveness/readiness probes on the deployed applications. This involves understanding how these components interact and making sure they are correctly set up from the first principles.

#### Triaging the Question

The question revolves around ensuring that the health checks reported by Argo CD accurately reflect the state of the application after deployment, particularly when readiness gates were not properly configured. Argo CD reports an application as "Healthy" if all its pods are ready, which can be misleading if the application itself is not fully operational.

Key Concepts:
Readiness Gates: These are Kubernetes features that control whether a pod is considered "ready" for traffic.
Liveness and Readiness Probes: These are health checks used by Kubernetes to determine when an application has started correctly (liveness) or if it needs to be restarted (readiness).

#### Assumptions
The Argo CD deployment uses Kubernetes resources.
We need to ensure that the application is fully operational before allowing traffic.

#### First Principles Solution

To make health checks accurate, we need to configure both liveness and readiness probes correctly. Here’s a step-by-step approach:

Understand Readiness Gates:

Configure the readinessProbe in your Kubernetes deployment to ensure that only pods that pass these probes are considered ready for traffic.
This can be done by adding annotations or labels on the deployment and specifying the readiness probe's configuration.

Configure Liveness Probes:

Use liveness probes to ensure that the application is still running and responding correctly when it fails (e.g., a 500 error).
These are typically configured in the deployment file with options like initialDelaySeconds, periodSeconds, etc.

Deploy Application Changes Safely:

Use Argo CD to apply changes incrementally, ensuring that each step is validated before moving on.
Integrate continuous validation steps in Argo CD’s workflow, such as CI/CD pipelines that run health checks post-deployment.

Monitor and Validate:

Set up monitoring tools (Prometheus, Grafana) to continuously monitor the application's health.
Use these tools to validate that both liveness and readiness probes are correctly configured and functioning as expected.

#### Implementation
Steps

Define Liveness and Readiness Probes in Kubernetes Manifests:

YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  template:
    spec:
      containers:
        - name: example-container
          image: example-image:latest
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /readiness
              port: 8080
            failureThreshold: 3
            periodSeconds: 10

Configure Argo CD to Use these Probes:

Ensure that the Argo CD application’s spec includes the correct references to your Kubernetes manifest.
Add any necessary annotations or labels on the Argo CD application.

Implement CI/CD Pipelines for Continuous Validation:

Integrate Jenkins, GitLab CI, or another CI/CD tool to run automated health checks after each deployment.
Use these tools to trigger alerts if liveness and readiness probes fail.

Monitor and Logging:

Set up logging and alerting mechanisms in your monitoring stack (e.g., Prometheus + Alertmanager) to notify teams of any issues.

#### Closing Summary

By configuring proper liveness and readiness probes, integrating continuous validation steps, and leveraging Argo CD’s capabilities, we can ensure that health checks accurately reflect the state of deployed applications. This approach not only improves user experience but also ensures the stability and reliability of the application in production.

#### Common Mistakes/Follow-Ups

Mistake: Failing to configure liveness probes properly.

#### Follow-Up
Always validate that the liveness probe correctly identifies when an application is no longer functional.

Mistake: Not configuring readiness gates or misconfiguring them.

#### Follow-Up
Ensure that pods are not considered ready until they pass all configured probes.

Mistake: Ignoring continuous validation and relying solely on Argo CD’s health checks.

#### Follow-Up
Implement a multi-layered approach to monitoring, including CI/CD pipelines, and integrate them with your monitoring tools.

**Comparison:**

```
Component	Description
Liveness Probe	Ensures the application is still running and responding correctly.
Readiness Probe	Ensures the application is ready to accept traffic.
Argo CD Sync Policy	Automates deployment and validation processes.
```

**Code:**

```yaml
# Example Kubernetes Deployment YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: example-app
spec:
  template:
    spec:
      containers:
        - name: example-container
          image: example-image:latest
          livenessProbe:
            httpGet:
              path: /healthz
              port: 8080
            initialDelaySeconds: 5
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /readiness
              port: 8080
            failureThreshold: 3
            periodSeconds: 10
---
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app-cd
spec:
  source:
    repoURL: https://github.com/example-repo
    path: app/deployment.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Example input / output:**

```text
Input:
- Deployment YAML with liveness and readiness probes configured.
- Argo CD application manifest with sync policy.

Output:
- Deployed application in Kubernetes with correct health checks setup.
- Argo CD reports accurate state of the application based on probe results.
- Continuous validation via CI/CD pipeline ensures post-deployment stability.
```

**Data:**

```json
{
  "source": {
    "repoURL": "https://github.com/example-repo",
    "path": "app/deployment.yaml"
  },
  "destination": {
    "server": "https://kubernetes.default.svc",
    "namespace": "production"
  }
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Is Application Deployed?
5. Configure Liveness/ReadinessProbes
6. Set Up CI/CD Pipeline forContinuous Validation
7. Deploy to Kubernetes withArgo CD
8. Resolve Deployment Issues
9. Monitor Application HealthUsing Prometheus/Grafana
10. Argo CD Syncs and ValidatesState
11. Maintain Configuration andTroubleshoot
12. Report Accurate Health Statusto Users
13. Ensure ContinuousImprovement

