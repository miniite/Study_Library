---
type: "practice"
difficulty: "Basic"
---

# If a bad alert rule got deployed and started paging everyone, what is the simplest way to undo it using version control?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Rollback Strategy](<../../../../10_DevOps_and_Platform/Delivery/CI_CD/Rollback_Strategy/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When an AIOps fresher encounters a situation where a misconfigured alert rule has been deployed and is causing unnecessary pings (or "pags"), reverting this change is crucial to prevent further disruption. The simplest way to undo such an issue using version control involves identifying the problematic commit, rolling it back, and ensuring everything returns to normal. Here's how you can approach this:

First, we need to understand that in a CI/CD pipeline, code changes are version-controlled using tools like Git. When a bad alert rule is deployed, there will be a specific commit (a change in the codebase) that introduced this issue.

Identify the Misleading Commit: First, you would navigate through your repository's history to locate the commit where the problematic code was merged. This could be done using Git commands like git log or by using an IDE's version control features.
Revert the Commit: Once identified, revert that specific commit using a rebase or cherry-pick. In Git, you can use the command git revert <commit-hash> to create a new commit that undoes the changes made in the problematic one.
Test and Verify: After reverting the commit, push your changes back to the repository and ensure that the pipeline re-deploys the application successfully without triggering unnecessary alerts.

By following these steps, you can effectively address the issue and return the system to its stable state.

