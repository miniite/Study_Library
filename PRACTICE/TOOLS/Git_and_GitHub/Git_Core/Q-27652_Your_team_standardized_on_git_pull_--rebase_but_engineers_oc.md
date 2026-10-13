---
type: "practice"
difficulty: "Deep"
---

# Your team standardized on git pull --rebase, but engineers occasionally rewrite already-pushed alert-rule commits, orphaning the deploy history other on-call folks rely on. What layered guardrails (config, hooks, branch protection, CI) keep rebase strictly local-only for the alerting repo?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Protected Branches and Required Checks](<../../../../TOOLS/Git_and_GitHub/Git_Core/Protected_Branches_and_Required_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a scenario where engineers occasionally rewrite already-pushed commits using git pull --rebase in an AIOps team, we must ensure that such actions do not orphan the deploy history crucial for on-call engineers. To achieve this, we need to implement layered guardrails at multiple levels: configuration settings, Git hooks, branch protection rules, and continuous integration (CI) pipelines.

TL;DR:

Configuration: Set up git client configurations to enforce local-only rebase.
Hooks: Use pre-commit hooks to prevent pushing rewritten commits.
Branch Protection Rules: Enforce strict policies on protected branches in GitHub.
CI Pipelines: Integrate CI checks to detect and prevent pushes with conflicting changes.

