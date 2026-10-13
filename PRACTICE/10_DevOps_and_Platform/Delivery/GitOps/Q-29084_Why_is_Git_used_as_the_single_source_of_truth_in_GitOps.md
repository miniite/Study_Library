---
type: "practice"
difficulty: "Basic"
---

# Why is Git used as the single source of truth in GitOps?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Repo and Environment Layout](<../../../../10_DevOps_and_Platform/Delivery/GitOps/Repo_and_Environment_Layout/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In GitOps, Git serves as the single source of truth for infrastructure and application configurations. This ensures that all deployment processes are version-controlled, reproducible, and auditable. By using Git, teams can leverage its powerful branching, merging, and collaboration features to manage their operational state in a consistent and predictable manner.

Git is used in GitOps because it provides a robust system for managing and evolving infrastructure as code (IaC). Let's break down the key aspects:

Version Control: Git allows you to maintain multiple versions of your configurations, which is crucial for tracking changes over time.
Reproducibility: With Git, you can easily revert to previous states if something goes wrong, ensuring that deployments are reproducible and reliable.
Collaboration and Auditing: Multiple stakeholders can collaborate on the same codebase while maintaining a clear audit trail of all changes.

To illustrate, let's consider an example where we manage Kubernetes configurations using Git. We have two branches: main (where stable configurations live) and dev (for experimental or new features). By merging from dev to main, you ensure that the latest validated configurations are promoted to production.

#### Triage

The question is probing why Git, specifically, is chosen as the single source of truth in GitOps. We need to assume a basic understanding of version control systems and their benefits for DevOps practices. The core idea here is about leveraging Git's strengths for managing infrastructure and application configurations.

#### First-Principles Solution
Version Control: Every change made to infrastructure or applications should be tracked. Git allows you to maintain a history of these changes, making it easy to revert to previous states if needed.
Reproducibility: Deployments must be repeatable and consistent. Git ensures that the exact same configuration can be deployed multiple times with confidence.
Collaboration and Auditing: Multiple teams or individuals need to work on the same configurations. Git provides a clear, auditable history of changes.

To further illustrate, consider managing Kubernetes YAML files in Git:

Main Branch: Contains stable configurations used for production.
Dev Branch: Holds experimental or new configuration changes.
Merging from dev to main ensures that all validated changes are promoted to the production environment.

#### Closing Summary

In summary, using Git as the single source of truth in GitOps provides a robust solution for version control, reproducibility, and collaboration. This approach ensures that infrastructure and application configurations are managed with precision and accountability.

#### Common Mistakes/Follow-Ups
Ignoring Changes: Teams might overlook changes made through manual interventions outside the Git repository.
Complex Merge Conflicts: Handling complex merge conflicts in multi-environment setups can be challenging.
Security Risks: Sensitive information should not be committed to the main branch; proper access controls and encryption are necessary.

**Example input / output:**

```text
Input:
- Branch: dev
  Changes made in k8s-deployment.yaml

Output:
- Merged into main branch after resolving conflicts
- Updated `k8s-deployment.yaml` with new configuration
```

