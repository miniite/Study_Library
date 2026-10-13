---
type: "practice"
difficulty: "Deep"
---

# A canary rollout of a new model reads its traffic-split config from Git, but a mid-rollout revert to the previous commit left canary and stable inconsistent. How does a Git revert interact with an in-flight progressive rollout, and how do you abort cleanly?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverts in GitOps Rollouts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverts_in_GitOps_Rollouts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we need to understand how Git reverts affect ongoing deployments and ensure that any rollback is handled gracefully without disrupting the current state of the system. A canary rollout reads its traffic-split configuration from Git, but when a mid-rollout revert to an earlier commit was performed, it left the canaries in an inconsistent state with respect to stable traffic.

To begin, let's clarify what’s happening:

The new model is being deployed progressively.
Git commits represent different states of the model and its configuration.
A git revert operation undoes a previous commit but does not change history; instead, it introduces a new commit that effectively reverts the changes.

#### Triaging the Question
What’s Probed: The interaction between deployment stages and version control system operations.

#### Assumptions
The canary rollout is reading its configuration from Git directly.
A git revert was executed during an ongoing canary rollout, affecting both stable and canary traffic.

#### First Principles Solution

Understanding the Canary Rollout:

Canary rollouts typically involve a small subset of users being served by the new model while the majority still use the old version.
The traffic split is dynamically managed, often via configuration files stored in Git or other version-controlled systems.

Impact of git revert:

A git revert introduces a new commit that undoes changes made by an earlier commit but doesn’t alter the history linearly. This means the system might still see old configurations due to cached versions.
If not properly managed, this can lead to inconsistencies between stable and canary traffic.

Handling Rollback Gracefully:

Identify the exact commit that introduced the issue.
Use git revert again to undo the revert if necessary.
Ensure that all environments (canaries and stable) are updated consistently by redeploying the correct configuration.
Step-by-Step Solution

Identify the Issue:

Check Git history to pinpoint which commit introduced the inconsistency.
Verify that the canary traffic is still using the reverted version of the model.

Execute a Clean Revert:

Use git revert on the commit that caused the issue, ensuring it’s done in a safe environment first (e.g., staging).
Once verified, apply to production, but ensure this doesn’t affect ongoing rollouts.

#### Cons
istent Deployment Strategy:

If possible, use CI/CD pipelines that automatically fetch and deploy the latest configuration from Git.
Implement checks to prevent stale configurations in flight (e.g., checksum validation).

Graceful Abortion:

Develop a rollback plan that can be triggered if an issue is detected during deployment.
Use git reset carefully, understanding its implications on history and states.

#### Closing Summary

A clean rollback strategy involves identifying the exact commit causing issues, reverting safely in stages, and ensuring all environments are updated consistently. Proper CI/CD practices can automate these steps, reducing manual errors and increasing reliability.

#### Common Mistakes

Failing to understand the git revert mechanism and its implications.
Not verifying changes across all environments before fully deploying them.
Overlooking the need for thorough testing in staging environments.

#### Follow-Up
Questions
How do you ensure canaries and stable traffic are updated simultaneously?
What tools or practices do you use to validate that Git configurations are correctly applied?
Can you explain how to integrate a rollback plan into your CI/CD pipeline?

