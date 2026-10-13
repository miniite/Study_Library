---
type: "practice"
difficulty: "Expert"
---

# How do you wire a pipeline to watch metrics for a few minutes post-deploy and auto-abort if they degrade?

**Difficulty:** Expert · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Deployment Observability](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To wire up a CI/CD pipeline that watches metrics post-deployment and automatically aborts if they start degrading, we need to design a robust system that can handle various edge cases while ensuring reliability and maintainability. At an expert level, the key ideas involve using monitoring tools, implementing delay logic, and setting up alerting mechanisms.

The core of this solution involves several steps:

Post-deployment Monitoring: Set up a script or tool to monitor metrics right after deployment.
Alerting Mechanisms: Define thresholds for acceptable metric values and trigger alerts if these are crossed.
Automated Rollback: Implement a mechanism that can automatically rollback the deployment if metrics start degrading.

Let's break down each step in detail:

#### Triaging the Question

The question is essentially asking how to set up post-deployment monitoring with an automated rollback feature to prevent degraded performance from persisting. The interviewer is probing your understanding of CI/CD pipelines, monitoring tools, and the ability to implement complex logic that involves delay, threshold checks, and conditional rollbacks.

#### Assumptions
We are working in a Kubernetes environment.
Monitoring tools like Prometheus or Grafana are available.
Deployment is managed using GitOps with tools like Argo CD or Flux.

#### Solution
from First Principles
1. Post-deployment Monitoring

We can use a custom script to collect and monitor metrics for a few minutes post-deployment. This script will be run as part of the CI/CD pipeline after deploying new code or configuration changes.

BASH
#!/bin/bash

# Assuming we have a Kubernetes deployment named 'my-app'
NAMESPACE=my-app-namespace

# Define the duration in seconds to wait before monitoring starts (e.g., 180 seconds)
WAIT_TIME=180

# Wait for the application to start up and become ready
sleep $WAIT_TIME

# Collect initial metrics using kubectl or a custom script that interacts with the service
initial_metrics=$(kubectl get pods -n $NAMESPACE | grep Running | wc -l)

# Monitor the system continuously (for example, every 10 seconds)
while true; do
    current_metrics=$(kubectl get pods -n $NAMESPACE | grep Running | wc -l)

    if [ "$current_metrics" -lt "$initial_metrics" ]; then
        echo "Degradation detected: Number of running pods decreased."
        # Trigger rollback mechanism
        trigger_rollback
        break
    fi

    sleep 10  # Wait for 10 seconds before checking again
done
2. Alerting Mechanisms

Set up alert rules in Prometheus or another monitoring tool to notify the team if certain metrics start degrading. For example, you can create a rule that triggers an alert when the number of failed pods exceeds a threshold.

YAML
# Example alert rule in Prometheus (prometheus-rules.yaml)
groups:
- name: my-app-degradation
  rules:
  - alert: MyAppPodsDegradation
    expr: kube_pod_status_phase{namespace="my-app-namespace", phase="Failed"} > 0
    for: 5m
    labels:
      severity: critical
    annotations:
      summary: "My app pods are degrading."
3. Automated Rollback

Implement a rollback mechanism using the same CI/CD tool (e.g., Argo CD) to revert to a previous stable state if degradation is detected.

YAML
# Example rollback strategy in Argo CD
resources:
  - name: my-app-deployment
    group: apps
    version: v1
    kind: Deployment
    namespace: my-app-namespace
    apiVersion: argoproj.io/v1alpha1
    metadata:
      annotations:
        argocd.argoproj.io/last-sync-time: "{{ .LastSyncTime }}"
    spec:
      template:
        spec:
          containers:
            - name: my-app-container
              image: <previous-image-version>

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, the solution involves setting up a custom script to monitor post-deployment metrics, configuring alert rules in Prometheus to trigger alerts if necessary, and implementing an automated rollback strategy using CI/CD tools.

#### Common mistakes
include:

Not considering edge cases like network disruptions or delays in deployment.
Failing to set appropriate thresholds that could lead to false positives.
Ignoring the need for proper logging and documentation of the rollback process.

#### Follow-up
questions might involve discussing how to handle different types of metrics (e.g., response times, error rates), scaling this solution for larger systems, or integrating with existing monitoring stacks.

**Code:**

```

#!/bin/bash

Custom script for post-deployment monitoring and rollback
CODE
Copy

```yaml
# Example Prometheus alert rule configuration
YAML
Copy
# Example Argo CD deployment specification for automated rollback
```

**Example input / output:**

```text
Input:
- Deployment of 'my-app' in namespace 'my-app-namespace'
- Custom script to monitor post-deployment metrics

Output:
- Alert triggered if number of running pods decreases by more than a certain threshold.
- Automated rollback to previous stable state if degradation is detected.
```

**Data:**

```json
{
  "namespace": "my-app-namespace",
  "wait_time_seconds": 180,
  "initial_metrics": {
    "running_pods": 5
  },
  "alert_rule": {
    "expr": "kube_pod_status_phase{namespace=\"my-app-namespace\", phase=\"Failed\"} > 0",
    "for": "5m"
  }
}
```

