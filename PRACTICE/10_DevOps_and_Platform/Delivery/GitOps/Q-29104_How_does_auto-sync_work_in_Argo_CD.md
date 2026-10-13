---
type: "practice"
difficulty: "Intermediate"
---

# How does auto-sync work in Argo CD?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Argo CD vs Flux](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Auto-sync in Argo CD is a critical feature that ensures your application state on Kubernetes clusters matches the desired configuration defined in Git repositories, effectively implementing GitOps principles. In this context, "auto-sync" refers to the automatic process of applying changes from Git into the cluster's actual state. Let’s break down how it works and why it’s important.

#### Triaging the Question

Argo CD uses a combination of Git repositories as sources of truth for application configurations and Kubernetes clusters as targets where these configurations are applied. The key idea here is that Argo CD ensures that the desired state (defined in Git) matches the actual cluster state through continuous reconciliation.

#### First Principles Solution

To understand auto-sync, we need to look at how Argo CD works from first principles:

Source of Truth: Argo CD uses a Git repository as its source of truth for application definitions.
Application Resource Definitions: These are Kubernetes manifests or Helm charts stored in the Git repository.
Reconciliation Loop: Argo CD continuously monitors these resources and applies any necessary changes to bring the cluster state into alignment with the desired configuration.
Detailed Explanation

Git Repository Setup:

Define your application’s configurations in a Git repo, e.g., https://github.com/your-org/my-app.
Use Argo CD to sync this repository with your Kubernetes cluster.

Application Definitions:

Create Helm charts or YAML manifests in the Git repo that define the desired state of your applications.
These definitions include everything from deployments, services, and CRDs (Custom Resource Definitions) to roles and role bindings.

Reconciliation:

Argo CD periodically checks the Git repository for changes.
For each application definition, it creates a corresponding Application resource in Kubernetes.
The Application resource tracks the desired state from Git and the current cluster state.
If there are discrepancies (i.e., the cluster doesn't match the desired state), Argo CD applies the necessary patches to bring them into alignment.

Sync Strategy:

You can configure sync strategies such as sync-only, prune-and-sync, etc., which determine how changes should be applied.
For instance, sync-only ensures only new or updated resources are applied without deleting existing ones.

Health Checks and Notifications:

Argo CD provides health checks to ensure the application is running correctly.
It also sends notifications via integrations like Slack or email when there are issues.

#### Closing Summary

In summary, auto-sync in Argo CD ensures that your Kubernetes cluster state matches the desired configuration stored in Git. This process involves continuous monitoring and applying changes as needed. The key steps include setting up a Git repository with application definitions, defining reconciliation strategies, and leveraging Argo CD’s built-in features for health checks and notifications.

#### Common Mistakes/Follow-Ups
Not Configuring Sync Strategies Properly: Incorrect sync strategies can lead to unnecessary deletions or missing resources.
Ignoring Health Checks: Failing to monitor the health of applications can result in undetected issues.
Overlooking Notification Settings: Neglecting to set up notifications can delay issue resolution.

Argo CD’s auto-sync is a powerful tool for maintaining consistency between your Git repositories and Kubernetes clusters, making it easier to manage complex multi-cluster environments with minimal manual intervention.

**Comparison:**

```
Reconciliation Strategy	Description
sync-only	Only new or updated resources are applied. No existing resources are deleted.
prune-and-sync	Existing resources not present in Git are removed, then the Git configuration is synced.
```

**Code:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/your-org/my-app.git
    targetRevision: main
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
- Git repository URL: https://github.com/your-org/my-app.git
- Target revision: main

Output:
- Application created in Kubernetes cluster.
- Sync policy applied to ensure desired state is maintained.
- Health checks and notifications enabled for monitoring.
```

**Data:**

```json
{
  "repoURL": "https://github.com/your-org/my-app.git",
  "targetRevision": "main",
  "namespace": "production",
  "syncStrategy": {
    "automated": {
      "prune": true,
      "selfHeal": true
    }
  }
}
```

**Flow:**

1. Git Repository
2. Application Definition
3. Kubernetes Cluster
4. Desired State
5. Actual State
6. Reconciliation Loop
7. Health Checks & Notifications

