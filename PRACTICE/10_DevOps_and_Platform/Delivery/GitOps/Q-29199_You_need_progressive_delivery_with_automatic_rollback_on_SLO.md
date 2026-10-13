---
type: "practice"
difficulty: "Expert"
---

# You need progressive delivery with automatic rollback on SLO breach — how do you wire Argo Rollouts metric analysis to your golden signals?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Sync Notifications](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Sync_Notifications.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enable progressive delivery with automatic rollback based on Service Level Objectives (SLO) breaches in a GitOps environment, we need to integrate metric analysis from Argo Rollouts into our deployment pipeline and correlate it with golden signals. This ensures that we can make informed decisions about when to proceed or roll back deployments.

#### Triaging the Question

The key here is understanding the integration points between Argo Rollouts' metrics and GitOps practices, specifically focusing on SLO breaches for rollback automation. The core idea is to leverage Argo Rollouts' powerful features while ensuring that our deployment processes are aligned with our operational goals as defined by golden signals.

#### Assumptions
Golden Signals: These include latency, traffic trends, error rates, and saturation levels.
Argo Rollouts: A Kubernetes native continuous delivery platform for deploying new application versions to production in a reliable way.
GitOps: Practices based on Git as the single source of truth for infrastructure configuration.
SLOs (Service Level Objectives): Performance targets agreed upon by teams and stakeholders.

#### Solution
Set Up Argo Rollouts:
Configure Argo Rollouts to track metrics related to your golden signals using Prometheus or another monitoring system.
Define SLOs in Argo Rollouts:
Use rollout objects within Argo Rollouts to define the desired state and thresholds for each metric. For example, you might set an error rate threshold of 1%.
Integrate Metrics with GitOps:
Use GitOps tools like Flux or Argo CD to ensure that the deployment configurations are version-controlled and sync these metrics back into the Git repository.
Automate Rollback on SLO Breach:
Utilize Argo Rollouts' rollback capabilities to automatically revert to a previous successful rollout if any of your defined SLOs are breached.
Step-by-Step Solution
Configure Argo Rollouts for Metrics Collection:
YAML
apiVersion: argoproj.io/v1alpha1
kind: RollingUpdateStrategy
metrics:
  - name: error-rate
    threshold: 0.01
    type: percentile
    sourceRefName: prometheus-server
    query: rate(http_request_errors[5m])
Define SLOs and Rollback Policy:
YAML
apiVersion: argoproj.io/v1alpha1
kind: RollingUpdateStrategy
rollbackPolicy:
  enabled: true
  criteria:
  - name: error-rate
    threshold: 0.01
    type: percentile
    sourceRefName: prometheus-server
    query: rate(http_request_errors[5m])
Integrate Metrics with GitOps:
Use kubectl commands or a CI/CD pipeline to push metric data into the Git repository.
Ensure that Argo Rollouts can read metrics from this repository and apply them as part of the deployment process.

#### Closing Summary

By integrating Argo Rollouts' metric analysis capabilities with your GitOps workflows, you can achieve robust progressive delivery with automated rollbacks. This approach ensures that any SLO breaches are handled seamlessly, maintaining service reliability and compliance with agreed-upon operational targets.

#### Common Mistakes/Follow-Ups
Failing to Define Clear Metrics: Ensure all relevant metrics (golden signals) are properly defined.
Misconfigured Rollback Policies: Incorrect thresholds or missing criteria can lead to unnecessary rollbacks.
Overcomplicating the Solution: Keep the integration straightforward and align with your GitOps practices.

**Comparison:**

```
Define Argo Rollout Metrics
Configure PrometheusServiceMonitor
Deploy Argo Rollouts withmetrics
Integrate metrics into GitOpsworkflows
Implement rollback policy inArgo Rollouts
Rollback triggered on SLObreaches
```

**Code:**

```

apiVersion: argoproj.io/v1alpha1 kind: RollingUpdateStrategy metrics:

name: error-rate threshold: 0.01 type: percentile sourceRefName: prometheus-server query: rate(http_request_errors[5m])
CODE
Copy
```yaml
apiVersion: argoproj.io/v1alpha1
kind: RollingUpdateStrategy
rollbackPolicy:
  enabled: true
  criteria:
  - name: error-rate
    threshold: 0.01
    type: percentile
    sourceRefName: prometheus-server
    query: rate(http_request_errors[5m])
```

**Example input / output:**

| Step | Action             | Expected Outcome |
|------|--------------------|-----------------|
| 1    | Deploy Argo Rollouts with metrics configuration | Metrics collected and SLOs defined |
| 2    | Integrate metrics into GitOps workflows      | Metrics pushed to Git repository   |
| 3    | Implement rollback policy in Argo Rollouts  | Rollback triggered on SLO breaches |

**Data:**

```yaml
# Example configuration for Prometheus metrics
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: example-prometheus-monitor
spec:
  endpoints:
  - port: web
    interval: 30s
---
# Example Argo Rollout configuration
apiVersion: argoproj.io/v1alpha1
kind: RollingUpdateStrategy
metrics:
  - name: error-rate
    threshold: 0.01
    type: percentile
    sourceRefName: prometheus-server
    query: rate(http_request_errors[5m])
---
# Example GitOps configuration
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
spec:
  source:
    repoURL: https://github.com/your-repo.git
---
# Example rollback policy in Argo Rollouts
apiVersion: argoproj.io/v1alpha1
kind: RollingUpdateStrategy
rollbackPolicy:
  enabled: true
  criteria:
  - name: error-rate
    threshold: 0.01
    type: percentile
    sourceRefName: prometheus-server
    query: rate(http_request_errors[5m])
```

**Flow:**

1. No
2. Yes
3. Start Deployment
4. Argo Rollout MetricsCollection
5. SLO Breach Detected?
6. Proceed to Next Stage ofDeployment
7. Rollback Triggered by ArgoRollouts
8. End Deployment with CorrectState

