---
type: "practice"
difficulty: "Deep"
---

# How do you architect GitOps for a multi-tenant SaaS where each customer gets an isolated namespace and config, but the base platform is shared?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-tenant GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/Multi-tenant_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

In this architecture, we'll leverage GitOps to ensure that each tenant in a multi-tenant SaaS environment can have their own isolated namespace and configuration while sharing a common base platform. The key is to maintain consistency across all tenants using a central repository with templating and overlays.

#### Triage

The question is asking how to architect a GitOps approach for managing configurations across multiple tenants, each in their isolated namespaces, ensuring that the shared platform remains consistent yet allows for customizations. The core aspects are:

Namespace Isolation: Each tenant should have an independent namespace.
Shared Base Platform: Core functionalities and infrastructure components are shared among all tenants.
Configuration Management: Tenant-specific configurations need to be managed separately but consistently applied.
Step-by-Step Solution
1. Architecture Overview

We'll use a multi-tenant architecture where each tenant has their own Kubernetes namespace, and we leverage GitOps principles for configuration management. The shared base platform will include common components like the database cluster, API servers, and network services. Tenant-specific configurations (e.g., environment variables, secret keys) will be managed via overlays applied to these namespaces.

2. Central Repository and Workflows
Central Repository: A Git repository where we store all the base platform configurations and tenant-specific configuration files.
Branching Strategy: Use a main branch for the shared base platform code, and branches or tags for different tenants' specific configurations.
3. Kubernetes Namespace Management
Namespace Creation: Create Kubernetes namespaces dynamically based on tenant IDs using CI/CD pipelines or API calls.
Role-Based Access Control (RBAC): Ensure that only authorized users can access and modify their respective namespaces and overlays.
4. Configuration Templates and Overlays
Base Configurations: Store the base platform configurations in a Git repository, such as k8s/ for Kubernetes configurations.
Tenant-Specific Templates: Create templates or overlays for each tenant's specific configurations stored separately but aligned with the base configurations.
5. Continuous Deployment and Syncing
CI/CD Pipelines: Use CI/CD tools like Jenkins, GitLab CI, or GitHub Actions to automate deployment processes.
Kustomize (or Helm): Utilize tools like Kustomize for applying overlays over the base configurations dynamically during deployment.
6. Monitoring and Logging
Central Monitoring: Set up centralized logging and monitoring using solutions like Prometheus, Grafana, and ELK stack to monitor all namespaces.
Alerting: Configure alerts based on thresholds and metrics that are relevant across all tenants.

#### Closing Summary

In summary, the key components of this GitOps architecture include a central repository for base configurations, dynamic namespace management, tenant-specific overlays, and continuous deployment pipelines. The challenge lies in maintaining consistency while allowing for customization by leveraging Kustomize or similar tools to manage overlays effectively. Common mistakes include not properly isolating namespaces, failing to enforce RBAC, and lacking robust monitoring and alerting.

#### Common Mistakes/Follow-Ups
Failure to Properly Isolate Tenants: Ensuring that tenant configurations do not interfere with each other is crucial.
RBAC Misconfiguration: Failing to set up appropriate roles and permissions can lead to unauthorized access and potential security risks.
Lack of Monitoring and Alerting: Without proper monitoring, issues might go unnoticed until it’s too late.

**Comparison:**

```
Tenant	Namespace	Config Files Applied
A	tenant-a	base-configs, tenant-a.yaml
B	tenant-b	base-configs, tenant-b.yaml
```

**Code:**

```python
# Example CI/CD pipeline using GitLab CI
stages:
  - build
  - deploy

build_job:
  stage: build
  script:
    - kustomize build overlays/$TENANT | kubectl apply -f -

deploy_job:
  stage: deploy
  script:
    - kubectl rollout restart deployment/app-1
```

**Example input / output:**

```text
Tenant A configuration applied:
- Namespace: tenant-a
- Deployment: app-1
- Patches applied from: patch-overlays/tenant-a.yaml

Tenant B configuration applied:
- Namespace: tenant-b
- Deployment: app-2
- Patches applied from: patch-overlays/tenant-b.yaml
```

**Data:**

```yaml
# Example tenant-specific overlay in Kustomize
resources:
- ./base-configs
patchesStrategicMerge:
- patch-overlays/$TENANT.yaml
```

**Flow:**

1. CI/CD Trigger
2. Kustomize Build
3. Kubectl Apply
4. Kubernetes NamespaceCreation
5. RBAC Check
6. Tenant Specific Config Applied

