---
type: "practice"
difficulty: "Intermediate"
---

# How does GitOps make it easier to answer the question who changed what and when in production?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ GitOps Fundamentals](<../../../../10_DevOps_and_Platform/Delivery/GitOps/GitOps_Fundamentals/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

GitOps leverages version control systems (VCS) like Git to manage infrastructure as code, making it much easier to track changes, rollbacks, and audit who made what changes when in production. By automating the deployment process with tools like Flux or Argo CD, GitOps ensures that all changes are recorded in a single source of truth—your VCS repository.

#### Triaging the Question

The question is essentially asking how GitOps simplifies change management and auditing in production environments. Specifically, it probes your understanding of:

Infrastructure as Code (IaC): The practice of managing infrastructure via code.
Version Control Systems (VCS): Tools that allow tracking changes over time.
Continuous Deployment: Automated processes for deploying changes.
Audit Traces: Recording and reviewing changes made to the production environment.

#### Assumptions
We are using a VCS like Git for managing infrastructure configurations.
The deployment process is automated with tools like Flux or Argo CD.
The production environment is under version control, meaning every change is committed to a branch and tagged appropriately.

#### First-Principles Solution
Step 1: Understanding IaC

In traditional DevOps practices, infrastructure changes might be made directly on the physical servers, which can lead to a lack of traceability. GitOps transforms this practice by treating the infrastructure as code (IaC). This means that all infrastructure configurations are stored in version-controlled repositories.

Step 2: Automating Deployment with GitOps Tools

Tools like Flux or Argo CD automatically apply changes from your VCS repository to the production environment based on specific triggers, such as a push to a branch. These tools monitor the state of your application and infrastructure, ensuring that they match the desired configuration stored in the repository.

Step 3: Tracking Changes with Git

Every change made to the IaC files is committed to the VCS repository along with a commit message describing what was changed. This makes it easy to track who made the changes and why. For example:

Commit Message Example: "Updated DNS records for frontend service."
SHA Commit Identifier: abc123456
Step 4: Rollbacks and Auditing

If a deployment fails or causes issues, you can easily revert to a previous state by simply rolling back the last commit. This is done using Git commands like git reset --hard HEAD~1. Additionally, detailed logs of all changes are available in the VCS repository.

Step 5: Integration with CI/CD Pipelines

GitOps integrates seamlessly with CI/CD pipelines, ensuring that every change goes through a defined process before being applied to production. This includes code reviews, automated testing, and other quality checks.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, GitOps makes it easier to answer "who changed what and when" by:

Centralizing Configuration: All infrastructure configurations are stored in version-controlled repositories.
Automating Deployments: Tools like Flux or Argo CD automatically apply changes based on triggers.
Tracking Changes with Git: Every change is logged with commit messages and SHA identifiers.

#### Common mistakes
to avoid include:

Failing to automate deployments, leading to manual interventions that can introduce errors.
Not using detailed commit messages, making it hard to trace who made what changes.
Ignoring the integration of CI/CD pipelines, which ensures a disciplined approach to change management.

#### Follow-up
questions might include how you handle security and compliance in GitOps environments or your experience with specific tools like Flux or Argo CD.

**Comparison:**

```
Tool	Key Features	Example Use Case
Flux	Real-time monitoring, automated rollbacks	Monitor and apply IaC changes to production clusters
Argo CD	CI/CD integration, visual dashboard	Deploy applications using GitOps principles with continuous delivery pipelines
```

**Code:**

```bash
# Example commit message
git commit -m "Updated DNS records for frontend service"
```

**Example input / output:**

| Input (Commit Message) | Output (Git Commit) |
|------------------------|--------------------|
| Updated DNS records for frontend service. | abc123456          |

**Data:**

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  source:
    repoURL: https://github.com/myorg/my-infrastructure-repo.git
    path: infrastructure
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
```

**Flow:**

1. Commit Changes
2. Push to Repository
3. Trigger CI/CD Pipeline
4. Apply Changes in Production
5. Monitor and Rollback ifNecessary

