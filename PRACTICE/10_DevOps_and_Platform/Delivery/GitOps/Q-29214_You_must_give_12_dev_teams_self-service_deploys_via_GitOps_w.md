---
type: "practice"
difficulty: "Deep"
---

# You must give 12 dev teams self-service deploys via GitOps without letting them touch each other's namespaces — design the Argo CD Project and repo permissions.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Multi-tenant GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/Multi-tenant_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that 12 dev teams can perform self-service deploys using GitOps without interfering with each other’s Kubernetes namespaces, we need to carefully design the architecture of our Argo CD projects and repository permissions. This approach will allow teams to have fine-grained control over their deployments while maintaining isolation between them.

#### Triage
Understanding the Problem

The core challenge is to enable multiple dev teams to deploy applications into different Kubernetes namespaces without any risk of namespace pollution or accidental deployment conflicts. GitOps provides a framework for managing the state of infrastructure and applications using version-controlled manifests, but we need to ensure that each team's access is strictly controlled.

#### First-Principles Solution
Namespace Isolation: Each dev team will have its own Kubernetes namespace.
Project Structure in Argo CD: We'll create one GitOps project per namespace to manage the application configurations for that specific namespace.
Repository Permissions: We'll set up appropriate repository access controls to ensure each team can only modify their respective namespaces.
Detailed Implementation
Step 1: Namespace Isolation

We will configure 12 distinct Kubernetes namespaces, one for each dev team:

YAML
apiVersion: v1
kind: Namespace
metadata:
  name: team-01
---
apiVersion: v1
kind: Namespace
metadata:
  name: team-02
...
Step 2: Project Structure in Argo CD

For each namespace, we will create a corresponding GitOps project in Argo CD. Each project will be responsible for deploying and managing the application manifests within that specific namespace.

YAML
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: team-01-app
  namespace: team-01
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: team-01
  source:
    repoURL: git@github.com:our-repo/team-01.git
...
Step 3: Repository Permissions

We will set up GitLab, GitHub, or any other version control system to host the repositories. For each repository, we will configure access controls as follows:

ReadOnly Access: All dev teams should have read-only access to shared libraries and common templates.
Full Control Access: Each team will have full control over their respective application manifests.

#### Example
Repository Structure
CODE
git@github.com:our-repo/
├── team-shared-library
│   ├── manifests
│   └── templates
└── team-applications
    ├── team-01.git
    │   ├── manifests
    │   └── templates
    └── ...

#### Closing Summary
and Common Mistakes/Follow-Ups

#### Closing Summary

By designing the Argo CD projects to correspond directly to namespaces and setting up granular repository permissions, we ensure that each dev team can perform self-service deploys without interfering with others. This approach also allows for easy scaling as more teams join or leave the project.

#### Common Mistakes/Follow-Ups
Mistake 1: Not enforcing strict namespace isolation may lead to accidental deployments.
Mistake 2: Overly permissive repository permissions can result in unauthorized changes.

#### Follow-Up
Additional security measures, such as regular audits and automated deployment validation, should be implemented.

**Comparison:**

```
Namespace	Application Repository
team-01	git@github.com:our-repo/team-01.git
team-02	git@github.com:our-repo/team-02.git
```

**Code:**

```yaml
# Example Argo CD Application Manifests
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: team-01-app
  namespace: team-01
spec:
  destination:
    server: https://kubernetes.default.svc
    namespace: team-01
  source:
    repoURL: git@github.com:our-repo/team-01.git
```

**Example input / output:**

```text
Input: Team "team-01" pushes a new deployment manifest to their repository.
Output: Argo CD monitors the change, reconciles the state with the desired configuration in namespace "team-01".
```

**Data:**

```text
# Namespace Manifests
apiVersion: v1
kind: Namespace
metadata:
  name: team-01
---
apiVersion: v1
kind: Namespace
metadata:
  name: team-02
...
```

**Flow:**

1. Yes
2. No
3. Repository Update
4. Is the change in a sharedlibrary?
5. Audit and Notify
6. Argo CD Monitors Change
7. Reconcile State with DesiredConfig in Namespace

