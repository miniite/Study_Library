---
type: "practice"
difficulty: "Intermediate"
---

# How would you keep your staging alerting rules from paging real on-call, while still deploying and exercising them through the same pipeline as prod?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Artifact Promotion Across Environments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Artifact_Promotion_Across_Environments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the challenge of keeping staging alerting rules from pinging on-call engineers during deployment and testing, we need to ensure that staging environments have similar but isolated configurations compared to production. This involves implementing a combination of technical controls and best practices in our CI/CD pipeline. The key steps involve creating separate namespaces or environments for staging, using dynamic environment-specific alerts, and ensuring proper access control.

**Code:**

```python
# Example configuration for alerting rules
staging_alerts = {
    "monitor": "cpu_usage",
    "threshold": 90,
    "action_on": "warn"
}

production_alerts = {
    "monitor": "cpu_usage",
    "threshold": 95,
    "action_on": "critical"
}
```

**Example input / output:**

```text
graph TD;
    A[Staging Environment] --> B(Configure Alert Rules)
    C[Production Environment] --> D(Use Production Rules)
    E[Pipeline Trigger] --> F[Deploy Staging]
    F --> G[Run Tests]
    G --> H[Update Staging Alerts]
    I[On-Call Rotation] --> J[Different Teams for Staging]
```

**Data:**

```json
{
  "staging": {
    "alert_rules": [
      {"name": "CPU Usage", "threshold": 90, "severity": "warning"}
    ]
  },
  "production": {
    "alert_rules": [
      {"name": "CPU Usage", "threshold": 95, "severity": "critical"}
    ]
  }
}
```

