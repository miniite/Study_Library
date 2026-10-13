---
type: "practice"
difficulty: "Deep"
---

# A team wants feature-branch preview environments spun up per PR via GitOps — how do you generate and garbage-collect ephemeral environments cleanly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Resilience to Git Outages](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

To enable feature-branch preview environments to be spun up per Pull Request (PR) using GitOps, we need a solution that dynamically creates and manages these environments while ensuring they are cleaned up properly after their use. The key is to leverage GitOps principles, coupled with Kubernetes and an appropriate CI/CD pipeline.

#### Triage

The question requires understanding how to implement dynamic environment creation and garbage collection for feature branches in a GitOps setup. This involves using Kubernetes (or another orchestration tool), integrating with the Pull Request workflow, and ensuring that resources are managed efficiently.

#### Assumptions
The team uses Git for version control.
The CI/CD pipeline is already set up to trigger on PRs.
Kubernetes is used for managing environments.
We aim to keep the setup as automated as possible while maintaining control over resource allocation and cleanup.

#### First-Principles Solution
Environment Definition: Define each preview environment as a Kubernetes cluster or namespace with specific configurations.
CI/CD Integration: Use a CI/CD tool (e.g., Jenkins, GitLab CI) to trigger the creation of these environments based on PR events.
Resource Management: Implement garbage collection policies to ensure that unused environments are cleaned up automatically.
Step-by-Step Solution

Environment Definition:

Each feature branch should have a corresponding namespace or cluster in Kubernetes.
Use Helm charts for consistent deployment of application stacks, ensuring each environment has the same setup.

CI/CD Pipeline Setup:

Configure your CI/CD pipeline to listen for PR events and trigger a job specifically for creating preview environments.
The pipeline should include steps for:
Cloning the feature branch.
Applying Kubernetes manifests (Helm charts or custom YAML files) to create the environment.
Setting up any necessary configuration, such as service accounts and roles.

Dynamic Environment Creation:

Use kubectl commands or a CI/CD plugin to dynamically apply the Helm chart or Kubernetes manifest for each PR.

#### Example
helm install <chart-name> --namespace <feature-branch-name>.

Garbage Collection Policies:

Implement a regular cleanup job that runs periodically (e.g., daily) to identify and delete unused environments.
Use labels or annotations in Kubernetes to mark environments for deletion based on their age or last activity.

#### Example
A cronjob that checks namespaces/cluster names against the latest PRs and deletes those that are not recent.

Cleanup Script:

Write a script that can be triggered by the cleanup job to delete old environments.
The script should identify unused resources (pods, services, etc.) and gracefully shut them down before deletion.

#### Example
kubectl get pods --namespace <feature-branch-name> | awk '{print $1}' | xargs kubectl delete pod.

Monitoring and Alerts:

Set up monitoring to alert on failures or issues with environment creation or cleanup.
Use tools like Prometheus, Grafana, and Alertmanager for this purpose.

#### Closing Summary

By leveraging GitOps principles, we can ensure that feature-branch preview environments are dynamically created and managed in a controlled manner. This approach not only streamlines the development process but also ensures that resources are cleaned up promptly to avoid unnecessary costs and clutter.

#### Common Mistakes/Follow-Ups
Failure to Automate: Failing to fully automate the creation and cleanup processes can lead to manual errors and delays.
Insufficient Monitoring: Not setting up adequate monitoring and alerting can result in unnoticed issues with environment management.
Over-Provisioning: Creating too many environments without proper garbage collection can lead to resource wastage.
Security Risks: Ensuring that only authorized users or processes can trigger the creation of environments is crucial.

Would you like a more detailed example of how to set up these components? If so, I can provide an example CI/CD pipeline configuration and sample Kubernetes manifests.

**Comparison:**

```
Feature Branch	Age In Days	Resources To Delete
feature-branch-dev-123	7	Namespace, Deployments
feature-branch-dev-456	90	None
```

**Code:**

```
Example Helm Chart values.yaml
apiVersion: v1 kind: Namespace metadata: name: {{ .Values.namespace }}

apiVersion: apps/v1 kind: Deployment metadata: name: app-deployment spec: replicas: 3 selector: matchLabels: app: app-name template: metadata: labels: app: app-name spec: containers: - name: app-container image: <image-url>

CODE
Copy

```yaml
# Example Kubernetes namespace creation manifest
apiVersion: v1
kind: Namespace
metadata:
  name: feature-branch-dev-{{ .Values.branch }}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app-name
  template:
    metadata:
      labels:
        app: app-name
    spec:
      containers:
      - name: app-container
        image: <image-url>
BASH
Copy
# Example cleanup script (cleanup.sh)
#!/bin/bash

# Get list of namespaces with feature-branch names
FEATURE_BRANCHES=$(kubectl get ns | awk '{print $1}' | grep 'feature-branch-dev-' | sort)

for namespace in $FEATURE_BRANCHES; do
  # Check if the namespace is older than a certain threshold (e.g., 7 days)
  AGE=$(kubectl get namespaces "$namespace" -o jsonpath="{.metadata.creationTimestamp}")
  DAYS_OLD=$(( $(date +%s) - $(date --date="$AGE" +%s) / 86400 ))

  if [ $DAYS_OLD -gt 7 ]; then
    echo "Deleting namespace: $namespace"
    kubectl delete ns "$namespace"
  fi
done
```

**Example input / output:**

```text
# Example input (PR event)
git push origin feature-branch

# Expected output (CI/CD pipeline creates environment)
Environment namespace created for 'feature-branch-dev-123'
Deployment 'app-deployment' deployed in 'feature-branch-dev-123'

# Cleanup job output (daily cleanup run)
Namespace 'feature-branch-dev-456' deleted due to age
```

**Data:**

```json
{
  "featureBranches": [
    {
      "name": "feature-branch-dev-123",
      "ageInDays": 7,
      "resourcesToDelete": [
        {
          "kind": "Namespace",
          "name": "feature-branch-dev-123"
        }
      ]
    },
    {
      "name": "feature-branch-dev-456",
      "ageInDays": 90,
      "resourcesToDelete": []
    }
  ]
}
```

**Flow:**

1. Yes
2. Yes
3. No
4. CI/CD Pipeline Triggers
5. PR Event?
6. Create Environment(Namespace/Cluster)
7. Monitor and ApplyConfigurations
8. Cleanup Job Triggers
9. Environment Too Old?
10. Delete Resources (CleanupScript)
11. Cleanup Complete
12. Continue Monitoring

