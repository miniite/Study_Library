---
type: "practice"
difficulty: "Intermediate"
---

# How would you validate manifests before they are merged into a GitOps repo?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Resilience to Git Outages](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Reconciliation_Failures/Resilience_to_Git_Outages.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

Validating manifests before merging them into a GitOps repository is crucial to ensure that changes adhere to governance policies and avoid deployment failures. This process typically involves using tools like Flux or Argo CD, combined with custom pre-commit hooks and static code analysis. We'll use a combination of Git commands, CI/CD pipelines, and Kubernetes validation strategies to achieve this.

#### Triaging the Question

The core of the question revolves around ensuring that changes made to manifests (Kubernetes YAML files) are validated before they are merged into a GitOps repository like GitHub or GitLab. The interviewer is likely probing for your understanding of continuous integration and deployment (CI/CD) practices, GitOps workflows, and Kubernetes validation mechanisms.

#### Assumptions
We assume the team uses a CI/CD pipeline to automate this process.
The manifests are stored in a Git repository managed by tools like Flux or Argo CD.
The team has a set of predefined policies for validating manifests, such as syntax checks, security rules, and resource conflicts.
Step-by-Step Solution:

Understand the Workflow:

Developers make changes to their local copy of the manifest files.
Changes are committed and pushed to the Git repository.
A CI/CD pipeline is triggered upon push events.

Set Up Pre-commit Hooks:

Install a pre-commit hook in your Git client (e.g., pre-commit).
The hook will run a series of validation steps before allowing the commit to be pushed.

Run Static Code Analysis:

Use tools like yamllint for syntax checking.
Implement static analysis with Kubectl validate or custom scripts that check resource names, labels, and other metadata.

Integrate with CI/CD Pipelines:

Create a CI/CD pipeline using tools like Jenkins, GitHub Actions, GitLab CI, or CircleCI.
Configure the pipeline to run validation steps on each pull request (PR) before merging.

Use Kubernetes Admission Controllers:

Implement custom admission controllers using MutatingWebhookConfiguration and ValidatingWebhookConfiguration.
These controllers can perform real-time validation of incoming API requests or manifest changes.

Automate with GitOps Tools:

Use tools like Flux or Argo CD to continuously sync your manifests from the Git repository.
Configure these tools to trigger redeploys only when validated changes are detected in the repository.

Review and Approve PRs:

Ensure that code reviews include validation checks, as manual verification can catch issues that automated tools might miss.

#### Closing Summary

Validating manifests before merging them into a GitOps repo is essential for maintaining consistency, security, and operational efficiency. By integrating multiple layers of validation—pre-commit hooks, CI/CD pipelines, and Kubernetes admission controllers—we ensure that only valid changes are deployed to production. This process not only prevents errors but also aligns with best practices in DevOps.

#### Common Mistakes/Follow-ups
Failing to Use Pre-commit Hooks: Ensuring local development machines run the same validation steps can prevent issues from being introduced.
Over-relying on Automated Tools: Manual review and approval processes are still necessary to catch subtle or complex issues that automated tools might miss.
Neglecting Real-time Validation: Admission controllers should be configured to validate changes in real-time to ensure immediate feedback.

**Comparison:**

```
Validation Tool	Purpose
yamllint	Syntax checking for YAML files.
kubectl validate	Ensures manifests are valid according to Kubernetes schema.
```

**Code:**

```yaml
# Example pre-commit hook using yamllint
---
repos:
  - repo: https://github.com/njleclerc/yamllint
    rev: 1.34.0
    hooks:
      - id: yamllint
        language_version: "2.7.5"
```

**Example input / output:**

```text
# Example CI/CD Pipeline Triggered by Push Event
Input:
- Commit with modified manifests in branch 'main'
Output:
- Validation results and build status from Jenkins or GitLab CI

# Validation Steps
1. Run `yamllint` on modified files.
2. Validate manifest syntax using `kubectl validate`.
3. Check for any naming conflicts or security issues.

# Sample Build Output (Jenkins)
Started by GitHub push by user@example.com
[Pipeline] node
Running on Jenkins in /var/jenkins/workspace/my-app-main-PR123
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (validate-manifests)
[Pipeline] sh
+ yamllint -c .yamllint.yml app/deployments/*
...
```

**Data:**

```json
{
  "repository": {
    "name": "my-app",
    "branch": "main",
    "url": "https://github.com/myorg/my-app.git"
  },
  "pull_request": {
    "id": 123,
    "title": "Add new feature: user management",
    "status": "open"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Developer Commits Changes
6. Push Event Triggered
7. CI/CD Pipeline Runs
8. Run Pre-commit Hooks
9. Validation Passes, PR Approved
10. Fix Issues, Push Again
11. Run CI/CD Validation Steps
12. Validate with yamllint
13. Check Kubernetes Schemawith kubectl validate
14. Validation Passes
15. PR Approved for Merge
16. Fix Issues, Push Again

