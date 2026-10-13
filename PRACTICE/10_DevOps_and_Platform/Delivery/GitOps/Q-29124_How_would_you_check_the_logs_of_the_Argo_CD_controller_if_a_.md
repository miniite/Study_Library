---
type: "practice"
difficulty: "Intermediate"
---

# How would you check the logs of the Argo CD controller if a sync fails?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Scaling Argo CD](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/Scaling_Argo_CD.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To check the logs of the Argo CD controller when a sync failure occurs, we need to understand that GitOps involves ensuring that the desired state of our infrastructure is reflected in code and managed through version control systems like Git. When a sync fails, it's crucial to diagnose what went wrong by examining the Argo CD controller's logs.

The core idea here is to identify the root cause quickly and efficiently so we can resolve issues swiftly. We will first understand the environment setup and then walk through the steps to retrieve relevant logs.

**Code:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
spec:
  source:
    repoURL: https://github.com/example/repo.git
  destination:
    server: https://kubernetes.default.svc
    namespace: default
```

**Flow:**

1. Yes
2. No
3. Application Sync Starts
4. Controller Fetches Latest Code
5. Applies to Local State
6. Local State matches DesiredState?
7. Sync Successful
8. Reconciliation Loop Initiates
9. Reconcile and Log Errors
10. Log审查中...

