---
type: "practice"
difficulty: "Deep"
---

# Your deploy went out and error rates jumped. Walk me through your first five minutes.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Troubleshooting 5xx Errors](<../../../../06_Cloud_Computing/Cloud_Foundations/Cloud_Fundamentals/Troubleshooting_5xx_Errors.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When a deployment goes wrong and error rates spike, my initial response is to quickly assess the situation, diagnose the issue, and then take steps to mitigate impact while identifying root causes. The first five minutes are critical for understanding what's happening.

In this scenario, I would immediately triage the issue by checking the following:

Environment: Confirm that the error is not confined to a single environment or instance.
Logs: Inspect the logs of the deployed service to identify any anomalies or failures.
Monitoring Alerts: Review monitoring alerts and dashboards to see if there are any red flags, such as unexpected metrics like CPU usage, memory consumption, or network latency.
Service Health: Check the health status of the service from a user perspective using tools like Prometheus and Grafana.

Let's break this down step by step:

#### Triage
Process

Step 1: Confirm Environment Impact

Check Logs: Log in to the deployment environment (e.g., Kubernetes cluster, cloud provider console) and review logs for any error messages or anomalies.
Use Metrics Dashboard: Look at metrics like HTTP requests, response times, and error rates on monitoring dashboards. This can help identify if the issue is isolated or widespread.

Step 2: Inspect Logs

Analyze Log Files: Use tools like kubectl logs for Kubernetes or cloud provider-specific logging tools to inspect recent log entries.
Search for Patterns: Look for patterns of errors, such as stack traces or error messages that indicate a specific failure mode.

Step 3: Review Monitoring Alerts

Alerts Dashboard: Check the monitoring dashboard (e.g., Prometheus + Grafana) for any unexpected spikes in metrics like latency, response time, and error rate.
Identify Specific Metrics: Look at specific metrics that are relevant to the service. For example, HTTP status codes or API request rates.

Step 4: Service Health

User Perspective: Use tools to test the service from a user's perspective (e.g., curl, Postman) to see if the issue is reproducible.
Service Mesh: If using a service mesh like Istio, check the traces and logs for any anomalies.
First Five Minutes Solution

First Action: Confirm Environment

BASH
kubectl get pods -n <namespace>
# Check pod status and logs
kubectl logs <pod-name> -n <namespace>

Second Action: Review Metrics

Open the monitoring dashboard and check for any anomalies.

#### Example
query in Prometheus:
PROMQL
rate(http_requests_total[1m])
# Check if error rates are increasing

Third Action: Analyze Logs

Search for relevant log entries to understand the issue.
Use log aggregation tools like Elasticsearch and Kibana for more detailed analysis.

#### Conclusion
and Follow-Ups

In the first five minutes, I would have a clear understanding of whether the error is isolated or widespread. This initial triage will guide my next steps—whether it's rolling back the deployment, scaling resources, or investigating code changes that might be causing the issue.

#### Common mistakes
to avoid include:

Not checking logs and metrics before making assumptions.
Ignoring user perspective in favor of backend-only checks.
Failing to document findings and communicate with stakeholders.

