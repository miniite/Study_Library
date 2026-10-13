---
type: "practice"
difficulty: "Deep"
---

# A self-hosted runner keeps picking up jobs from forked PRs. Why is that dangerous and how do you architect runner groups to prevent it?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Self-hosted Runners](<../../../../TOOLS/Git_and_GitHub/GitHub_Actions/Self-hosted_Runners.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Self-hosted GitHub runners are often used in CI/CD pipelines, but allowing them to pick up jobs from forked pull requests (PRs) can introduce significant security and operational risks. This scenario is dangerous because it exposes your self-hosted runners to potential external threats, such as malicious code or untrusted repositories. To mitigate these risks, you need to carefully architect your runner groups.

To understand why this is a problem, let's start with the basics: GitHub Actions use self-hosted runners to execute jobs defined in workflows. These runners can be configured to run on specific branches, forks, or even pull requests. If a runner is set up to pick up jobs from any PR, it may inadvertently execute code from untrusted sources.

The key steps to architecting secure runner groups involve:

Identifying the types of workloads and their security requirements.
Isolating runners based on trust levels.
Implementing strict access controls.
Monitoring and logging.

Let's break this down further:

#### Triage
Understanding the Question

The question is asking why self-hosted GitHub runners picking up jobs from forked PRs poses a danger and how to prevent it. The core idea here is to ensure that your CI/CD pipeline remains secure by controlling who can execute what code on which runners.

#### First-Principles Solution
Define Runner Groups: Create distinct runner groups based on the trustworthiness of repositories.
Repository-Level Controls: Ensure that only trusted repositories are allowed to create PRs and trigger workflows.
Branch Protection Rules: Implement strict branch protection rules to control what can be merged into your main branches.
Security Policies for Self-Hosted Runners: Configure self-hosted runners to run only in secure environments, such as private networks or isolated VMs.
Step-by-Step Solution

Define Runner Groups:

Create a group of runners dedicated to trusted repositories (e.g., your own development team’s repositories).
Create another group for external contributors who may occasionally submit PRs (though even this should be carefully monitored).

Repository-Level Controls:

Use GitHub settings to restrict which repositories can create or modify pull requests.
Implement two-factor authentication and other security measures.

Branch Protection Rules:

Apply strict branch protection rules to main branches, requiring approvals from multiple maintainers before merging.
Disable direct pushes to these branches, ensuring that changes only come through PRs.

Security Policies for Self-Hosted Runners:

Ensure runners are isolated and cannot access sensitive networks or data.
Use network segmentation to prevent lateral movement if a runner is compromised.
Regularly update and patch the runners to protect against vulnerabilities.

#### Closing Summary

By carefully architecting your GitHub Actions setup, you can significantly reduce the risk of executing untrusted code on self-hosted runners. This involves defining secure runner groups, implementing strict access controls, and ensuring that all components of your CI/CD pipeline are well-protected.

#### Common Mistakes/Follow-Ups
Failing to Isolate Runners: Not properly separating trusted and untrusted repositories can lead to accidental execution of malicious code.
Insufficient Monitoring: Failing to set up adequate logging and monitoring can make it difficult to detect when a runner has been compromised.
Lack of Regular Updates: Neglecting to regularly update and patch self-hosted runners increases the risk of vulnerabilities being exploited.

**Comparison:**

```
Define Secure Groups
Repository-Level Controls
Branch Protection Rules
Security Policies for Runners
```

**Code:**

```yaml
# Example GitHub Actions configuration for secure runner groups

name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
    paths: [ '*.py' ] # Protect specific files or directories

jobs:
  build:
    runs-on: self-hosted-trusted
    steps:
      - name: Checkout repository
        run: |
          echo "Checking out trusted repository..."
      - name: Run tests
        run: python test.py

  review:
    runs-on: self-hosted-untrusted
    steps:
      - name: Checkout pull request
        if: github.event.pull_request.head.repo.full_name == 'your-org/your-repo'
        run: |
          echo "Checking out untrusted repository..."
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Push to `main` branch | Runner from `self-hosted-trusted` group executes test suite. |
| Pull request to `main` branch from fork | No action, since only trusted repositories are allowed. |

**Data:**

```json
{
  "repository": {
    "name": "example-repo",
    "owner": "user123",
    "forked": false,
    "hasIssuesEnabled": true,
    "isPrivate": true
  },
  "branch_protection_rules": [
    {
      "pattern": "main",
      "required_pull_request_reviews": {
        "require_code_ownerReviews": false,
        "dismiss_stale_reviews": false,
        "required_status_checks": {
          "context": ["build"]
        }
      },
      "restrictions": []
    }
  ]
}
```

**Flow:**

1. +---------------------+
2. | Define Groups       |
3. | (Trusted/Untrusted) |
4. +--------+-----------+
5. |
6. Yes ---+
7. |
8. +--------+-----------+
9. | Implement Rules   |
10. | for Repositories  |
11. +--------+-----------+
12. |
13. Yes ---+
14. |
15. +--------+-----------+
16. | Set Up Branch    |
17. | Protection       |
18. | Rules            |
19. +--------+-----------+
20. |
21. Yes ---+
22. |
23. +--------+-----------+
24. | Secure Runners   |
25. | with Network     |
26. | Isolation        |
27. +-------------------+

