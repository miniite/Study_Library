---
type: "practice"
difficulty: "Intermediate"
---

# Why is GitOps considered more secure than giving everyone kubectl access?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Secrets in GitOps](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Secrets_in_GitOps.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In this answer, we'll explore why GitOps is often considered more secure compared to granting kubectl access to everyone. We’ll start by defining what GitOps and kubectl are, then discuss common security challenges in Kubernetes clusters when using kubectl. Next, we'll delve into the principles of GitOps and how it mitigates these risks through version control, automation, and separation of concerns.

#### Triaging the Question

The question is probing your understanding of Kubernetes security practices. It assumes you know that kubectl provides direct command-line access to Kubernetes clusters, which can be risky if not managed properly. The core of the answer lies in explaining how GitOps enhances security by leveraging version control and automated workflows.

#### Assumptions
You understand what kubectl is and its role in managing Kubernetes resources.
You know basic concepts of GitOps, such as using Git repositories to manage cluster state and ensuring immutable infrastructure changes through CI/CD pipelines.

#### First-Principles Solution

Why GitOps is More Secure:

Version Control: In GitOps, all desired states are stored in a Git repository, typically managed by a version control system like GitHub or GitLab. This means that any change to the cluster's configuration must be made through approved pull requests (PRs). PRs provide a formal mechanism for reviewing and approving changes before they take effect.

Automated Workflows: GitOps relies on CI/CD pipelines to enforce automated validation, testing, and deployment processes. These pipelines can include security checks, such as running linters to ensure that new configurations adhere to best practices or automated tests to verify the correctness of updates.

Immutable Infrastructure: GitOps promotes immutable infrastructure principles where changes are treated as state transitions rather than modifications. This means that deploying a change is akin to creating a new snapshot in a version control system, ensuring that historical states can be rolled back if needed without disrupting existing cluster operations.

RBAC and Access Control: With GitOps, access controls can be more granular because changes are managed through Git repositories rather than direct kubectl commands. Fine-grained role-based access control (RBAC) policies can be enforced to restrict who can propose or approve changes, reducing the risk of unauthorized access.

Auditability and Traceability: Every change in a GitOps setup is recorded in the version history of the repository. This provides an audit trail that makes it easier to track down exactly what was changed, by whom, and when. This transparency helps in maintaining compliance with security policies and regulations.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, GitOps enhances security through its reliance on version control, automated workflows, immutable infrastructure principles, granular access controls, and auditability. However, it's important to implement these practices correctly; common mistakes include not properly securing the repository itself or neglecting to enforce strong CI/CD policies.

#### Follow-Ups

How would you set up RBAC in a GitOps environment?
Can you provide an example of a security check that might be included in a CI/CD pipeline for a GitOps workflow?

**Comparison:**

```
Resource	Kubernetes Access	GitOps
Change Control	kubectl commands directly	Version-controlled PRs with approvals
Automation and Testing	Manual deployment	CI/CD pipelines for automated testing and validation
Security Checks	Limited to pre-deployment or post-deployment	Pre-flight checks as part of the build process
```

**Code:**

```yaml
# Example of a GitHub Action for pre-deployment checks
on:
  push:
    branches:
      - main

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v2

    - name: Run security scan
      run: npx semgrep --config .semgrep.yaml .

    - name: Lint YAML files
      run: yamllint .
```

**Example input / output:**

```text
Input:
- New configuration changes proposed in a GitOps repository.

Output:
- PR created with automated tests and security checks.
- If successful, the change is merged into the cluster via an automated deployment pipeline.
- If not, the PR remains open for review or is closed if issues are found.
```

**Data:**

```json
{
  "repo": {
    "name": "example-app",
    "url": "https://github.com/example/example-app"
  },
  "pull_request": {
    "title": "Add new feature",
    "number": 42,
    "created_at": "2023-10-01T12:00:00Z",
    "state": "open"
  }
}
```

**Flow:**

1. New Configuration Changes
2. version-controlled PRs
3. Approval Process
4. Automated DeploymentPipeline
5. Merged into Cluster
6. Rejected for Fixes

