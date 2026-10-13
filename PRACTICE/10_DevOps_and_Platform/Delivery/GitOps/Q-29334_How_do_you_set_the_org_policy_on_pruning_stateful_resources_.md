---
type: "practice"
difficulty: "Deep"
---

# How do you set the org policy on pruning stateful resources so an accidental manifest deletion can never destroy a PVC or database, while keeping true declarative cleanup for stateless apps?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-2/3 · [↑ Resilience to Git Outages](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this GitOps challenge, we need to ensure that our GitOps pipeline adheres strictly to best practices for managing stateful resources like PersistentVolumeClaims (PVCs) and databases while also allowing for clean stateless application deletions. The key is setting org policies that prevent accidental deletion of critical resources without compromising the overall declarative cleanup capabilities.

#### Triaging the Question

The question asks about setting up GitOps policies to prevent accidental deletion of stateful resources such as PVCs and databases, while maintaining the ability to delete stateless applications cleanly through declarative cleanup. The core issue is balancing security with operational flexibility. We'll need to ensure that any deletion policy does not accidentally remove essential stateful components.

#### Assumptions
GitOps Workflow: We use a GitOps workflow where code changes are pushed to a git repository and automatically deployed.
Kubernetes Cluster: The cluster is managed using tools like GitOps controllers (e.g., Argo CD, Flux).
Stateful vs Stateless Resources: Stateful resources have unique identifiers (like PVCs) that need protection from accidental deletion.
Step-by-Step Solution

Identify Critical Resources:

Define a list of stateful applications that require special handling.

Organize the GitOps Repository:

Create separate directories for stateful and stateless resources within the repository.
Use .gitignore to ensure certain files are excluded from version control.

Apply Resource-Level Policies:

Leverage Kubernetes Admission Controllers (e.g., OpenShift's OPA policy) or GitOps controllers like Argo CD with custom policies.
Define a mutatingWebhookConfiguration that enforces pre-deletion checks on stateful resources.

Implement Custom Webhooks:

Create webhooks that run as part of the GitOps deployment pipeline to validate deletions.
YAML
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: stateful-resource-protection
webhooks:
- name: stateful-resource-protection.example.com
  rules:
  - operations: [ "DELETE" ]
    apiGroups: [ "*" ]
    apiVersions: [ "*/*" ]
    resources: [ "pvc", "statefulsets", "databases" ] # Custom API group for databases
  sideEffects: None
  clientConfig:
    service:
      namespace: default
      name: webhook-service
    caBundle: <base64-encoded-certificate>
  rulesMatchPolicy: All

Ensure Declarative Cleanup:

Use Helm charts or declarative YAML files for stateless resources to allow for clean deletions.
Ensure that the GitOps controller is configured to handle stateless resources with appropriate cleanup policies.

Configure RBAC and Role Policies:

Define fine-grained roles in Kubernetes and Git repositories to restrict access to critical resources.
YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: stateful-resource-manager
rules:
- apiGroups: [""]
  resources: ["pvc", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]

Test and Validate:

Perform thorough testing to ensure that stateful resources are protected from accidental deletion.
Conduct dry-runs with both stateful and stateless resources.

#### Closing Summary

By implementing a combination of resource-level policies, custom webhooks, and fine-grained RBAC, we can effectively protect stateful resources while maintaining the ability to perform clean deletions for stateless applications. This approach ensures that our GitOps pipeline is robust against accidental deletion events without compromising operational flexibility.

#### Common Mistakes/Follow-Ups
Mistake: Overly restrictive policies that inadvertently block legitimate changes.

#### Follow-Up
Ensure all stakeholders are involved in policy development and testing.
Mistake: Failing to implement thorough validation checks.

#### Follow-Up
Regularly review and update webhook configurations as new resources or workflows emerge.
Mistake: Not involving security best practices early in the design phase.

#### Follow-Up
Engage security experts to audit and validate policies.

**Code:**

```yaml
# Example MutatingWebhookConfiguration
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: stateful-resource-protection
webhooks:
- name: stateful-resource-protection.example.com
  rules:
  - operations: [ "DELETE" ]
    apiGroups: [ "*" ]
    apiVersions: [ "*/*" ]
    resources: [ "pvc", "statefulsets", "databases" ] # Custom API group for databases
  sideEffects: None
  clientConfig:
    service:
      namespace: default
      name: webhook-service
    caBundle: <base64-encoded-certificate>
  rulesMatchPolicy: All
```

