---
type: "practice"
difficulty: "Expert"
---

# How would you design the pipeline so that a rollback also reverts the observability/alerting config that shipped with the change, so on-call isn't blinded by mismatched dashboards after a rollback?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Rollback Strategy](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this CI/CD challenge, we need to ensure that both our application code and observability/alerting configurations are consistently managed across deployments and rollbacks. The goal is to keep the on-call engineers informed with accurate and relevant data post-rollback.

#### Triage

The question asks for a robust pipeline design that not only handles the rollback of application changes but also manages the corresponding observability and alerting configurations in unison. This means we need to ensure that when rolling back code, any associated monitoring dashboards and alerts are reverted to their previous state. The interviewer is probing your understanding of CI/CD principles, the integration of infrastructure as code (IaC), and the orchestration capabilities required for such a system.

#### First-Principles Solution

Identify Critical Components:

Application Code
Observability Configurations (Dashboards, Alert Rules)

Version Control:

Ensure both application code and observability configurations are version-controlled in a shared repository.

Configuration as Code:

Use tools like Helm charts or Kubernetes manifests to manage observability configurations.
Leverage IaC for consistent and repeatable deployments of these configurations.

CI/CD Pipeline Design:

Pre-Deployment Steps: Validate the changes and ensure they pass all necessary tests before proceeding.
Deployment Steps: Apply both application code and observability configurations.
Rollback Steps: Ensure that the rollback process also reverts both the application code and its associated observability configurations.

Rollback Strategy:

Maintain a history of deployments in the CI/CD pipeline.
Implement conditional logic to revert to previous states if necessary.
Detailed Solution

Repository Structure:

PLAINTEXT
repository/
├── app-code/
│   └── (application code)
├── observability-configs/
│   └── (observability configuration files, e.g., dashboards and alert rules in JSON/YAML format)
├── .gitignore
└── pipeline/
    └── (CI/CD pipeline scripts)

Pipeline Configuration:

Use Jenkins or GitLab CI to define the pipeline.
Stages for code validation, deployment of application and observability configurations.

Deployment Script Example:

BASH
# Deploy Application Code and Observability Configurations
echo "Deploying app version 1.0"
kubectl apply -f app-code/1.0/deployment.yaml
echo "Applying observability config v1.0"
helm upgrade --install observability observability-configs/v1.0/

Rollback Script Example:

BASH
# Rollback Application Code and Observability Configurations
echo "Rolling back app version 1.0 to 0.9"
kubectl rollout undo deployment/<app-name> --to-revision=9
echo "Reverting observability config v1.0 to v0.9"
helm upgrade --install observability observability-configs/v0.9/

History Management:

Use Git tags or version numbers to track deployments.
Ensure that each deployment is tagged and can be rolled back to a specific state.

#### Closing Summary

By integrating application code, observability configurations, and rollback strategies into a cohesive CI/CD pipeline, we ensure that both the application and its monitoring remain in sync. This approach not only simplifies the rollback process but also ensures that on-call engineers are always informed by accurate data.

#### Common Mistakes/Follow-Ups
Mistake: Failing to version control observability configurations.

#### Follow-Up
How would you handle cross-cluster or multi-environment rollbacks?

**Comparison:**

| Step | Action           | Notes                          |
|------|------------------|--------------------------------|
| 1    | Deploy application code & observability config     | Ensure both are version-controlled |
| 2    | Validate changes pass tests                       | Use pre-deployment validation steps  |
| 3    | Apply configurations to Kubernetes                 | Use Helm for consistency             |
| 4    | Maintain rollback history                          | Tag versions and use Git for tracking|
| 5    | Rollback application & observability as needed      | Ensure both revert to previous state |

**Code:**

```
Example deployment script

echo "Deploying app version 1.0" kubectl apply -f app-code/1.0/deployment.yaml echo "Applying observability config v1.0" helm upgrade --install observability observability-configs/v1.0/

CODE
Copy

```bash
# Example rollback script
echo "Rolling back app version 1.0 to 0.9"
kubectl rollout undo deployment/<app-name> --to-revision=9
echo "Reverting observability config v1.0 to v0.9"
helm upgrade --install observability observability-configs/v0.9/
```

**Example input / output:**

```text
Input: Deploy new application version and associated observability configurations.

Output: Application deployed successfully, observability dashboards updated accordingly.

Input: Rollback to previous application version and associated observability configurations due to issues in the latest deployment.

Output: Application rolled back to previous version 0.9, observability dashboards reverted to their state from version 0.8.
```

**Data:**

```yaml
# Example Helm chart for observability configuration
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: dashboard-pvc
spec:
  storageClassName: standard
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi

---
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: observability-dashboard-monitor
spec:
  selector:
    matchLabels:
      app: observability-dashboards
  endpoints:
    - port: web
      path: /metrics
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Code changes
7. Test pass?
8. Deploy app code &observability configs
9. Fix issues, repeat test
10. Rollback needed?
11. Rollback app & observability
12. End

