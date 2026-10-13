---
type: "practice"
difficulty: "Intermediate"
---

# How would you detect and alert on 'apps that have been OutOfSync for more than 15 minutes' across a fleet — what metrics does Argo CD expose?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Scaling Argo CD](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/Scaling_Argo_CD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of detecting and alerting on applications that have been OutOfSync for more than 15 minutes in a GitOps setup using Argo CD, we need to understand the metrics and mechanisms that Argo CD exposes and utilize them effectively.

#### Triaging the Question

The core issue here is ensuring real-time visibility into the synchronization status of your deployed applications across all clusters managed by Argo CD. Specifically, you want to monitor whether any application has diverged from its desired state for more than 15 minutes. This requires setting up a monitoring and alerting system that leverages the metrics provided by Argo CD.

Key Metrics in Argo CD

Argo CD exposes several key metrics through Prometheus:

argocd_app_last_update: The timestamp of the last update to an application.
argocd_app_last_sync: The timestamp when the application's state was last synced with the desired state.
argocd_app_status_synced: A boolean metric indicating whether the application is currently in a synchronized state.
Step-by-Step Solution

Identify OutOfSync Applications:

We need to monitor the argocd_app_last_update and compare it with argocd_app_last_sync. If there’s a significant difference, we have an OutOfSync application.

Define the Time Threshold:

Any application where argocd_app_last_update minus argocd_app_last_sync is greater than 15 minutes should be flagged.

Set Up Prometheus Queries:

PROMQL
(argo_app_status_synced == 0) and (time() - argocd_app_last_sync) > 900

This query checks for applications that are not in a synchronized state (argocd_app_status_synced is 0) and have exceeded the 15-minute threshold.

Configure Alerts:

Use Prometheus Alertmanager to create alerts based on this query.
Define alert rules like:
YAML
groups:
  - name: argocd-outofsync-alerts
    rules:
      - alert: ApplicationOutOfSync
        expr: (argo_app_status_synced == 0) and (time() - argocd_app_last_sync) > 900
        for: 5m
        labels:
          severity: warning
          app: "{{ $labels.app }}"
        annotations:
          summary: "Application {{ $labels.app }} is OutOfSync"
          description: "The application has not been synchronized with the desired state for more than 15 minutes."

#### Closing Summary
and Common Mistakes

By setting up these Prometheus queries and alerts, you ensure that any instance where an application diverges from its desired state by more than 15 minutes is detected promptly. This proactive monitoring helps in maintaining the integrity of your applications deployed across multiple clusters.

#### Common Mistakes

Not accounting for time zones or clock skew between different clusters.
Ignoring the argocd_app_status_synced metric, which can provide additional context on application synchronization status.
Misinterpreting the Prometheus query logic, leading to false positives or negatives in alerts.

**Code:**

```

argo_app_status_synced == 0 and time() - argocd_app_last_sync > 900

CODE
Copy
```yaml
groups:
  - name: argocd-outofsync-alerts
    rules:
      - alert: ApplicationOutOfSync
        expr: (argo_app_status_synced == 0) and (time() - argocd_app_last_sync) > 900
        for: 5m
        labels:
          severity: warning
          app: "{{ $labels.app }}"
        annotations:
          summary: "Application {{ $labels.app }} is OutOfSync"
          description: "The application has not been synchronized with the desired state for more than 15 minutes."
```

