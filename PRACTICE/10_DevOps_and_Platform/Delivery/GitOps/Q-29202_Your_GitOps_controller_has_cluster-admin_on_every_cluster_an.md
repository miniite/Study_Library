---
type: "practice"
difficulty: "Expert"
---

# Your GitOps controller has cluster-admin on every cluster and security flagged it as a blast-radius risk — how would you scope its RBAC down without breaking reconciliation?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-cluster Ordering](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/Multi-cluster_Ordering.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we have a GitOps controller that currently has cluster-admin privileges on all clusters. However, security flagged it as having an overly broad blast radius. To mitigate this risk while ensuring continuous delivery and reconciliation processes continue to function correctly, we need to scope down the RBAC (Role-Based Access Control) permissions without disrupting existing workflows.

Triaging the Problem

The core of the question is about refactoring our GitOps controller's RBAC so that it operates with minimal necessary privileges. The key aspects are:

Preserving Reconciliation: Ensuring that the controller can still perform its duties in managing and updating cluster resources.
Reducing Risk: Limiting the permissions to only those required for reconciliation, thus reducing potential blast radius.

#### Assumptions
We assume a Kubernetes environment with GitOps practices in place.
The GitOps controller is responsible for applying configurations from a Git repository to clusters.
We need to maintain the integrity and reliability of the deployment process.
Step-by-Step Solution
1. Identify Minimal Required Permissions

First, we identify which actions are essential for the GitOps controller to perform:

Read Access: To read the configuration files and ensure they exist in the repository.
Apply/Update: To apply or update resources based on changes in the repository.

In Kubernetes, these actions map to specific verbs: get, watch, list for reading; and create, update, patch, delete for applying updates.

2. Define a Custom Role

We will define a custom role that includes only the necessary permissions:

YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: gitops-controller-role
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "watch", "list"]
- apiGroups: [""]
  resources: ["pods", "configmaps", "secrets"]
  verbs: ["create", "update", "patch", "delete"]
3. Apply the Role to the GitOps Service Account

Next, we apply this role to a dedicated service account that is used by our GitOps controller:

YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: gitops-controller-sa
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: gitops-controller-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: gitops-controller-role
subjects:
- kind: ServiceAccount
  name: gitops-controller-sa
  namespace: <namespace>
4. Update the GitOps Controller to Use the New SA

Finally, we update our GitOps controller to use this service account. This involves modifying any deployment or pod definition for the controller to specify serviceAccountName: gitops-controller-sa.

#### Closing Summary
and Common Mistakes

By following these steps, we ensure that the GitOps controller is scoped down to minimal necessary privileges, significantly reducing security risk while maintaining its core functionalities.

#### Common Mistakes
to Avoid:

Overly Broad Permissions: Retaining cluster-admin permissions unnecessarily.
Insufficient Logging: Not properly logging and auditing access attempts can lead to security breaches.
Inadequate Testing: Failing to thoroughly test the new RBAC configuration before deploying it in a production environment.

