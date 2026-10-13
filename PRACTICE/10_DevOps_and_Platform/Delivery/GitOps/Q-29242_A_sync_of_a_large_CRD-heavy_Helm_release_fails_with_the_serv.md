---
type: "practice"
difficulty: "Deep"
---

# A sync of a large CRD-heavy Helm release fails with 'the server could not find the requested resource' on first apply — why does ordering break here and how do you fix it?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Reconciliation Failures](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we're dealing with a GitOps-based infrastructure where Helm is used to manage Kubernetes resources via a CRD-heavy release. The issue at hand is that an initial sync fails due to a 'the server could not find the requested resource' error, which suggests there's some misalignment between the expected state of your cluster and its actual state. Let’s break down why this might be happening and how we can address it.

Triaging the Problem

The core question here is: Why does ordering matter in a GitOps setup with CRDs, and how do you ensure that resources are applied in the correct order to avoid resource resolution failures?

First, let's define some key concepts:

CRDs (Custom Resource Definitions): These extend Kubernetes with custom types.
Helm: A package manager for Kubernetes that helps manage the deployment of applications on a cluster.

When using Helm in a GitOps setup, resources are typically defined in charts and stored in version control. The sync process involves applying these definitions to the cluster. If there’s an issue where a resource is not found when it should be, it likely means one or more prerequisites were not applied correctly or in the correct order.

#### First-Principles Solution
Understand the Resource Hierarchy: CRDs often depend on each other, and their application sequence matters.
Identify Dependencies: Check which resources are prerequisites for others. For example, certain custom resources might require a particular API group to be available first.
Review Helm Chart Structure: Ensure that your Chart.yaml and requirements.yml files correctly define dependencies between different charts or CRDs.
Order of Operations: Apply resources in the correct order to avoid circular dependencies or resource resolution errors.
Step-by-Step Solution
List All Resources: Identify all custom resources involved in the Helm release.
Determine Dependencies: Create a dependency graph where each node is a resource, and edges represent prerequisites.
Apply Resources Sequentially: Ensure that before applying any resource X, all its dependencies are already applied to the cluster.
Use Helm’s Dry Run: Before actual application, run a dry-run with helm install --dry-run to identify any potential issues.
Practical Example

Suppose we have two CRDs: AppConfig and Deployment. Deployment might depend on AppConfig.

YAML
# AppConfig.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: appconfigs.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true

---
# Deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: mycontainer
          image: busybox
          args:
            - sh
            - -c
            - >
              while [ ! "$(kubectl get config.appconfigs.example.com myconfig)" ]; do sleep 1; done;

To ensure Deployment is applied after AppConfig, we need to:

SH
helm install --dependency-update . # Ensure dependencies are resolved first
helm install . # Apply the chart

#### Common Mistakes
and Follow-Ups
Overlooking Dependency Resolution: Failing to properly manage dependencies can lead to errors.
Improper Resource Versioning: Incorrectly managing versions of CRDs or their associated resources can cause issues.
Misconfigured Helm Charts: Not correctly setting up Chart.yaml and requirements.yml can result in dependency resolution failures.

To avoid these, always validate your setup with dry runs and ensure that dependencies are well-documented and managed.

#### Closing Summary

In conclusion, the key to fixing ordering issues in a GitOps setup involving CRDs is understanding the resource hierarchy, identifying dependencies, ensuring proper sequencing of applications, and using Helm tools effectively. By following these steps, you can prevent 'resource not found' errors and ensure smooth deployments.

**Comparison:**

```
Resource	Preceding Resources
AppConfig	None
Deployment	AppConfig
```

**Code:**

```yaml
# AppConfig.yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: appconfigs.example.com
spec:
  group: example.com
  versions:
    - name: v1
      served: true
      storage: true

---
# Deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
        - name: mycontainer
          image: busybox
          args:
            - sh
            - -c
            - >
              while [ ! "$(kubectl get config.appconfigs.example.com myconfig)" ]; do sleep 1; done;
```

**Example input / output:**

```text
# Dry-run
helm install --dependency-update .
# Actual installation
helm install .
```

**Data:**

```yaml
# Chart.yaml
name: my-app
version: 0.1.0

# requirements.yaml
dependencies:
- name: example-crd
  version: 0.1.0
  repository: https://example.com/charts
- name: deployment
  version: 0.2.0
  repository: https://example.com/charts
```

**Flow:**

1. Dependencies
2. AppConfig
3. Deployment
4. Dry-run
5. Apply Charts

