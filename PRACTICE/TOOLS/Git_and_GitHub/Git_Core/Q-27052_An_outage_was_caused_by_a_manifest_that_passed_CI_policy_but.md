---
type: "practice"
difficulty: "Deep"
---

# An outage was caused by a manifest that passed CI policy but violated an unwritten operational rule, and Git shows the change but not the reasoning. How would you enforce that risky infra PRs carry a documented rollout plan and rollback linked in the commit or PR?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that any infrastructure (infra) pull requests (PRs) include a documented rollout plan and rollback strategy, we need to enforce best practices within our Git and GitHub workflows. The goal is to make sure every PR not only passes CI but also includes detailed operational documentation before it can be merged.

#### Triage
What's Being Probed

The question probes your understanding of:

Git Workflow: How infra changes are managed.
Documentation Practices: Ensuring thorough and documented operations.
Enforcement Mechanisms: Implementing rules that prevent risky PRs from being merged without proper documentation.

#### Assumptions
You have a CI/CD pipeline in place for your infrastructure code.
Git is used to manage all infrastructure changes.
Your team uses GitHub or a similar platform for version control.
Step-by-Step Solution

Identify the Problem: The infra PR passed CI but violated an unwritten operational rule due to lack of documentation.

Document the Requirement: Define what constitutes a documented rollout plan and rollback strategy. This could include:

Detailed steps for deployment, including dependencies and pre-requisites.
Expected behavior during normal operation and failure conditions.
Steps for rolling back changes if necessary.

Integrate Documentation into PR Process:

Pre-Commit Hooks: Use pre-commit hooks to ensure that every commit includes a link or reference to the documented rollout plan and rollback strategy.
PR Templates: Create mandatory PR templates in GitHub that require contributors to include links to relevant documentation.
Automated Checks: Implement automated checks within your CI/CD pipeline to verify that these references exist and are valid.

Enforce Documentation During PR Review:

Reviewer Roles: Assign roles for reviewing PRs, ensuring that someone checks the presence of documented rollout plans and rollback strategies.
Notifications: Configure GitHub to notify reviewers if a PR is missing required documentation links.

Educate Team Members: Ensure all team members are aware of these new requirements and understand their importance in maintaining system stability.

Monitor and Iterate:

Continuously monitor the effectiveness of these changes.
Gather feedback from teams and iterate on the process as needed.

#### Closing Summary

To enforce that risky infra PRs carry a documented rollout plan and rollback strategy, we need to integrate documentation checks into our Git workflow using pre-commit hooks, PR templates, automated CI/CD checks, and reviewer roles. This approach ensures that every change is thoroughly documented before it can be merged, reducing the risk of operational outages.

#### Common Mistakes/Follow-Ups
Lack of Automation: Not integrating these requirements into automation tools like Git hooks or GitHub actions.
Insufficient Review Process: Relying solely on developer memory for documenting rollout plans and rollbacks.
Poor Communication: Failing to communicate the new requirements clearly across the team.

**Comparison:**

```
Step	Action
1	Define documentation requirements.
2	Implement pre-commit hooks.
3	Create PR templates.
4	Configure automated checks in CI/CD pipeline.
```

**Example input / output:**

```

Input:

DIFF
Copy
# Before Commit
- git add infra_changes.yaml

# After Adding Pre-commit Hook and Committing
+ git add infra_changes.yaml
+ git commit -m "Add infra changes with missing rollout plan"

Output (Expected):

CODE
Copy
Missing link to rollout plan. Add one before committing.
```

**Flow:**

1. Initiate PR
2. Include Link to Documentation
3. Commit Changes
4. Pre-Commit Hook ChecksDocumentation
5. Commit Blocked, AddDocumentation

