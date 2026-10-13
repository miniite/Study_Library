---
type: "practice"
difficulty: "Deep"
---

# A canary of a new upstream version behind the gateway looks healthy on p50 but the gateway's p99 doubled; how do you set up automated canary analysis (Argo Rollouts/Flagger) that reads gateway metrics and aborts on tail latency?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Gateway Config Testing and Rollout](<../../../../07_Distributed_Systems_and_Design/APIs/API_Gateway/Gateway_Config_Testing_and_Rollout.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To set up an automated canary analysis using Argo Rollouts or Flagger, we need to monitor key performance indicators (KPIs) like p99 latency and ensure that it does not exceed a certain threshold before promoting the new version. This involves configuring the canary deployment to continuously check gateway metrics and abort the rollout if tail latency breaches an acceptable limit.

#### Triaging the Question

The question is asking how to set up automated canary analysis where we use Argo Rollouts or Flagger, specifically focusing on monitoring and aborting based on the 99th percentile (p99) latency metric of the API Gateway. The current issue observed is that while p50 (median) health seems fine, p99 has doubled, indicating potential tail latency issues.

#### Assumptions
We have a working Argo Rollouts or Flagger setup.
The API Gateway metrics are available via Prometheus or another monitoring system.
The application under test is running alongside the production version with canary traffic split configured.
Step-by-Step Solution:
1. Define Canary Traffic Split

First, we need to set up a canary deployment that splits traffic between the current (production) and new (canary) versions of the backend service via the API Gateway.

YAML
apiVersion: argoproj.io/v1alpha1
kind: Canary
metadata:
  name: canary-deployment
spec:
  originalService:
    name: existing-backend-service
  revisionHistoryLimit: 5
  analysis:
    interval: "30s"
    threshold: 60s
  trafficRouting:
    percentTraffic: 10
2. Configure Prometheus to Collect Metrics

Next, ensure that your monitoring system (Prometheus) is collecting the necessary metrics for the canary deployment. Specifically, we need p99 latency data.

YAML
# Example Prometheus rule group
groups:
- name: canary-metrics
  rules:
  - alert: HighCanaryLatency
    expr: api_gateway_request_latency{quantile="0.99", service="canary-backend-service"} > 500ms
    for: 1m
    labels:
      severity: critical
    annotations:
      summary: "High canary request latency detected"
3. Integrate Metrics with Argo Rollouts/Flagger

We need to tie the Prometheus alert into the automated rollback mechanism.

Using Argo Rollouts:
YAML
apiVersion: argoproj.io/v1alpha1
kind: Canary
metadata:
  name: canary-deployment
spec:
  originalService:
    name: existing-backend-service
  revisionHistoryLimit: 5
  analysis:
    interval: "30s"
    threshold: 60s
    abortOnFailure: true
  trafficRouting:
    percentTraffic: 10
  rollbackPolicy:
    type: Canary
    canaryMetrics:
      metrics:
      - name: api_gateway_request_latency
        value: 500ms
Using Flagger:
YAML
apiVersion: flagger.app/v1beta2
kind: Canary
metadata:
  name: canary-deployment
spec:
  service: existing-backend-service
 分析:
   - name: api_gateway_request_latency
     value: 500ms
  progressDeadlineSeconds: 60s
4. Monitor and Troubleshoot

With the setup complete, continuously monitor the canary deployment for any issues. Ensure that if p99 latency exceeds the threshold of 500ms (or whatever limit you define), the canary rollout is automatically aborted.

#### Closing Summary

By setting up an automated canary analysis using Argo Rollouts or Flagger and configuring Prometheus alerts, we ensure that tail latencies are monitored. If the p99 latency breaches our defined threshold, the new version deployment will be rolled back to prevent degradation of user experience.

#### Common Mistakes
and Follow-Ups:
Mistake: Not defining clear thresholds for metrics like p99 latency.

#### Follow-Up
Define specific metric values that trigger immediate rollback based on business SLAs.
Mistake: Failing to properly route traffic between production and canary versions.

#### Follow-Up
Ensure proper traffic routing is configured in the API Gateway.
Mistake: Overlooking continuous monitoring of KPIs.

#### Follow-Up
Implement comprehensive monitoring using tools like Prometheus, Grafana, or other observability solutions.

**Code:**

```yaml
# Argo Rollouts example YAML
apiVersion: argoproj.io/v1alpha1
kind: Canary
metadata:
  name: canary-deployment
spec:
  originalService:
    name: existing-backend-service
  revisionHistoryLimit: 5
  analysis:
    interval: "30s"
    threshold: 60s
    abortOnFailure: true
  trafficRouting:
    percentTraffic: 10

# Flagger example YAML
apiVersion: flagger.app/v1beta2
kind: Canary
metadata:
  name: canary-deployment
spec:
  service: existing-backend-service
  analysis:
   - name: api_gateway_request_latency
     value: 500ms
```

