---
type: "practice"
difficulty: "Intermediate"
---

# What could go wrong if two people edit the cluster manually while GitOps is running?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Reconciliation Failures](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with a GitOps setup, manual edits to the Kubernetes cluster can lead to significant issues. Here’s what could go wrong and how we should handle it:

TL;DR

Manual edits to the cluster while GitOps is running can cause inconsistencies between your desired state (as defined in the configuration repository) and the actual state of the cluster. This discrepancy might result in unmanaged changes, leading to potential service outages or data loss.

#### Triaging the Question

The question revolves around understanding the impact of manual edits on a Kubernetes cluster managed via GitOps. Specifically, it probes our knowledge of how GitOps operates, the risks associated with manual interventions, and best practices for ensuring consistency and reliability in CI/CD pipelines.

#### First-Principles Solution

#### Assumptions
GitOps Setup: The environment uses a GitOps tool like Flux, Pulumi, or Argo CD to manage the cluster.
Manual Edits: Two users are making changes directly on the Kubernetes cluster without updating the configuration repository.
Step-by-Step Reasoning

Desired State vs. Actual State

In a GitOps setup, the desired state of the infrastructure is defined in a git repository (e.g., main branch). The CI/CD pipeline ensures that this desired state is applied to the cluster.
If two people manually edit resources like services or deployments directly on the cluster, they might be making conflicting changes. For example:
YAML
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376

Concurrent Changes

Manual edits can lead to race conditions, where the changes might be applied in an incorrect order or overlap.
For instance, User A updates a deployment's container image tag to v1 while User B simultaneously sets it to v2. This conflict will result in one change overriding the other.

Untracked Changes

Manual edits are not tracked by the GitOps tool. The tool might only apply changes from the git repository and ignore these untracked changes.
As a result, the cluster’s actual state diverges from the desired state defined in the configuration files. This can cause discrepancies such as:
PLAINTEXT
Actual State: Deployment "my-app" is running with image tag v2 (desired)
Desired State: Deployment "my-app" should run with image tag v1

Service Outages

These discrepancies can lead to service outages, security issues, or data corruption.
For example, a manual change might introduce an insecure configuration that could be missed by automated checks.

Rollback and Recovery

The GitOps tool’s ability to detect and roll back changes is compromised. Rolling back to the desired state requires additional steps like manually reverting changes in git or redeploying using the CI/CD pipeline.
This manual process can introduce further human error, increasing the risk of accidental data loss.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while GitOps ensures that the cluster’s actual state aligns with the desired state defined in your git repository, manual edits can disrupt this balance. To mitigate these risks:

Enforce Strict Pull/Commit Policies: Ensure all changes are made through the CI/CD pipeline.
Use GitOps Tools for Automation: Leverage tools like Flux to automatically apply changes and detect drift.
Educate Team Members: Train developers and operators on best practices for using GitOps tools.

#### Common mistakes
include:

Failing to enforce strict pull policies in the development environment.
Not monitoring cluster state regularly.
Ignoring warnings or alerts from GitOps tools about untracked changes.

#### Follow-ups
might involve discussing alternative strategies like Canary Deployments, Blue/Green Deployments, and proper change management processes.

**Comparison:**

```
Scenario	Result
Both increase replicas	Deployment with 5 replicas (unexpected)
Both decrease replicas	Deployment with 2 replicas (desired)
```

**Code:**

```yaml
# Example of a Kubernetes deployment manifest before conflict
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: MyApp
  template:
    metadata:
      labels:
        app: MyApp
    spec:
      containers:
      - name: my-app
        image: registry.example.com/my-app:v1
```

**Example input / output:**

```text
# Initial state (desired)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: MyApp
  template:
    metadata:
      labels:
        app: MyApp
    spec:
      containers:
      - name: my-app
        image: registry.example.com/my-app:v1

# Manual change (unexpected)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5 # Manually increased by one user
  selector:
    matchLabels:
      app: MyApp
  template:
    metadata:
      labels:
        app: MyApp
    spec:
      containers:
      - name: my-app
        image: registry.example.com/my-app:v1

# Reverting to desired state (after detection)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3 # Manually reverted by another user
  selector:
    matchLabels:
      app: MyApp
  template:
    metadata:
      labels:
        app: MyApp
    spec:
      containers:
      - name: my-app
        image: registry.example.com/my-app:v1
```

**Data:**

```plaintext
# Example of a manual change to the deployment manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 5 # Manually increased by one user
  selector:
    matchLabels:
      app: MyApp
  template:
    metadata:
      labels:
        app: MyApp
    spec:
      containers:
      - name: my-app
        image: registry.example.com/my-app:v1
```

**Flow:**

1. Yes
2. No
3. Manual Change Detected
4. Is Change Confirmed?
5. Apply Change via CI/CD
6. Revert to Previous State
7. Alert Team for ManualIntervention

