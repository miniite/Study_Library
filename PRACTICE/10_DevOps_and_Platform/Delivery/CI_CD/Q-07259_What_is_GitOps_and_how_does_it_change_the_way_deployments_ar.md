---
type: "practice"
difficulty: "Deep"
---

# What is GitOps and how does it change the way deployments are done?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Push vs Pull Deployments](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Deployment_Observability/Push_vs_Pull_Deployments.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

GitOps is a method of using infrastructure as code (IaC) to manage and deploy applications, ensuring consistency and automation in your deployment pipelines. Traditionally, CI/CD involves manual steps like approvals, checks, and deployments, but GitOps takes it further by treating the entire state of the application, including its configuration, as code stored in version control systems such as Git.

To fully understand how GitOps transforms traditional CI/CD practices, let's break down the core concepts and explore a practical example.

**Comparison:**

```
Traditional CI/CD	GitOps
Manual steps required for deployment	Fully automated deployments via IaC and Git repositories
Configuration inconsistencies possible due to manual intervention	Consistent state across all environments, as code changes are versioned and reviewed
Limited traceability of configuration history	Detailed audit logs and version control ensure full visibility into changes
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[CI/CD Pipeline] --> B{Is configuration managed?}
    B -- No --> C[Manual Deployments]
    B -- Yes --> D[Consistent State through IaC]
    D --> E[GitOps]
```

**Example input / output:**

```

Consider a scenario where you have an application deployed on Kubernetes clusters. Traditionally, developers might manually apply changes or use ad-hoc scripts to update the configuration and deploy new versions. With GitOps, you manage all configurations in a Git repository.

For instance, suppose you want to add a new environment variable EXAMPLE_SECRET with value abc123. You would:

Add this change to your config.yaml file in the Git repository.
Commit and push the changes: git commit -m "Add example secret" && git push.
Trigger the CI/CD pipeline, which updates the live cluster using a tool like Flux or Argo CD.

The output from the pipeline might look like this:

PLAINTEXT
Copy
Step 1: Cloning repository...
Cloning into 'example-repo'...

Step 2: Applying changes to live clusters...
kubectl apply -f config-map.yaml

Cluster applied successfully.
```

**Data:**

```yaml
# Example YAML file for GitOps
apiVersion: v1
kind: ConfigMap
metadata:
  name: example-config
data:
  config.yaml: |
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: example-deployment
    spec:
      replicas: 3
      selector:
        matchLabels:
          app: example
```

**Flow:**

1. Developers
2. Write IaC & Code
3. Commit & Push to Git Repo
4. CI/CD Pipeline Triggered
5. Apply Changes to Live Clusters
6. Liveness Checks andMonitoring
7. Merge Requests for Review
8. Rollback Mechanisms in Place

