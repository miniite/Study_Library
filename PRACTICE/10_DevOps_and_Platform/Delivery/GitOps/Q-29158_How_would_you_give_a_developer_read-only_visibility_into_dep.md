---
type: "practice"
difficulty: "Intermediate"
---

# How would you give a developer read-only visibility into deployments using Argo CD?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Argo CD vs Flux](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Argo_CD_vs_Flux/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To provide developers with read-only visibility into deployments using Argo CD, we need to set up appropriate GitOps workflows and permissions. This involves configuring Argo CD to sync with a Git repository containing our application configurations, and then setting up roles and policies in the CI/CD pipeline to restrict access while ensuring that developers can view deployment states.

#### Triage

The question is asking for a method to grant read-only access to developers for viewing deployments managed by Argo CD. The interviewer might be probing your understanding of GitOps principles, Kubernetes RBAC (Role-Based Access Control), and best practices in CI/CD pipelines. We will assume that the repository containing our application manifests is already set up and accessible.

Step-by-Step Solution
Understand GitOps Workflow: GitOps ensures that all configuration changes are recorded as diffs on a shared, immutable source of truth (like a Git repository). Argo CD watches this repository to apply those configurations to the live environment.
Set Up Repository Access: Ensure the developer has read-only access to the Git repository containing the application manifests. This can be done via GitHub/Bitbucket/GitLab settings for branch permissions and pull requests.
Configure Argo CD:
Sync Rules: Configure sync rules in your argocd-cm ConfigMap or directly through the CLI to specify which repositories, paths, and namespaces should be synced.
Application Manifests: Ensure that application manifests are stored securely but accessible via read-only Git credentials.
Kubernetes RBAC:
Create a service account with minimal permissions in your Kubernetes cluster for Argo CD operations.
Define roles (e.g., view) and role bindings to give developers read access without any write capabilities.
Full Solution

Set Up Repository Access:

PLAINTEXT
git clone <repository-url>

Configure Sync Rules in Argo CD:

YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
  namespace: argocd
spec:
  source:
    repoURL: <git-repo-url>
    path: apps/example-app/
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  project: default
  syncPolicy:
    automated:
      selfHeal: true

Create a Service Account and Roles in Kubernetes:

YAML
apiVersion: v1
kind: ServiceAccount
metadata:
  name: argo-viewer
  namespace: argocd

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: argocd
  name: view-argocd
rules:
- apiGroups: [""]
  resources: ["pods","services"]
  verbs: ["get", "watch", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: view-argocd-binding
  namespace: argocd
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: view-argocd
subjects:
- kind: ServiceAccount
  name: argo-viewer
  namespace: argocd

Grant Read-Only Access to Developers:

Ensure developers are added as members of a Git repository with read-only access.
Assign the argo-viewer service account to the appropriate developer roles in your Kubernetes cluster.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, providing read-only visibility into deployments using Argo CD involves setting up proper repository access, configuring sync rules, and implementing fine-grained RBAC. Common mistakes include granting excessive permissions or not properly isolating development environments from production. Follow-ups might involve discussing how to handle secrets securely, integrating with CI/CD pipelines for automated testing, or scaling these practices across multiple applications.

**Comparison:**

```
Role	Verbs
view-argocd	get, watch, list
Subject	Kind
argo-viewer	ServiceAccount
PLAINTEXT
Copy
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: view-argocd-binding
  namespace: argocd
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: view-argocd
subjects:
- kind: ServiceAccount
  name: argo-viewer
  namespace: argocd
```

**Code:**

```yaml
# Application Manifest Example
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: example-app
  namespace: argocd
spec:
  source:
    repoURL: https://github.com/example/repo.git
    path: apps/example-app/
  destination:
    server: https://kubernetes.default.svc
    namespace: staging
  syncPolicy:
    automated:
      selfHeal: true

# Kubernetes RBAC Example
apiVersion: v1
kind: ServiceAccount
metadata:
  name: argo-viewer
  namespace: argocd

---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: argocd
  name: view-argocd
rules:
- apiGroups: [""]
  resources: ["pods","services"]
  verbs: ["get", "watch", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: view-argocd-binding
  namespace: argocd
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: view-argocd
subjects:
- kind: ServiceAccount
  name: argo-viewer
  namespace: argocd
```

**Example input / output:**

```text
Input:
- Developer has read-only access to the GitHub repository.
- `argo-viewer` service account is created with minimal Kubernetes permissions.

Output:
- Developers can view application deployments and states via Argo CD UI or CLI without modifying any manifests.
- Secure and isolated environment ensuring developers do not have write access.
```

**Data:**

```plaintext
# Git Repository URL
https://github.com/example/repo.git

# Application Path in Repo
apps/example-app/
```

**Flow:**

1. Repository Access
2. Configure Sync Rules
3. Application Manifests
4. Service Account & RBAC
5. Read-Only View ofDeployments
6. Minimally Privileged ServiceAccounts
7. Secure and IsolatedEnvironment

