---
type: "practice"
difficulty: "Deep"
---

# A teammate force-pushed a shared long-lived model-integration branch and three engineers' experiment commits vanished from the tip. As on-call owner, recover the lost commits and design guardrails so it can't happen again.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Recovering Lost Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Recovering_Lost_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

When a teammate force-pushed to a shared long-lived branch, causing experiment commits by three engineers to be lost, recovering those commits is crucial. The task involves understanding Git operations and designing robust guardrails to prevent such issues in the future.

#### Triaging the Question
Understanding the Problem: A force-push on a shared repository overwrote recent changes, resulting in the loss of experiment commits.
Key Assumptions:
We have access to the Git history and branches involved.
The team uses GitHub for version control.
Step-by-Step Solution:

Identify the Force-Pushed Commit: Use git reflog on the shared branch to find the commit hash that was force-pushed, which overwrote the experiment commits.

Find the Lost Commits: Run git log --walk-reflogs or use tools like git fsck to identify missing commits.

Recover the Lost Commits:

Clone the repository locally if necessary.
Use git reflog to pinpoint where the force-push occurred and which commit it overwrote.
Recover lost commits using techniques such as git cherry-pick, git merge, or git revert.

Revert the Force-Push: If possible, revert the force-push with git reflog to restore the branch state before the overwrite.

Design Guardrails for the Future:

Implement Git hooks and policies.
Educate team members on best practices.
Use feature branches for experiments and merge them into master when ready.
Step-by-Step Example:

Identify the Force-Pushed Commit:

BASH
git reflog

Look for the commit hash of the force-push operation.

Find the Lost Commits:

BASH
git log --walk-reflogs | grep <lost_commit_sha>

Recover the Lost Commits: If you find a lost commit, cherry-pick it back into the branch.

BASH
git checkout <branch_name>
git cherry-pick <commit_hash>

Revert the Force-Push (if feasible):

BASH
git reflog expire --all && git gc --prune=now
git revert -m 1 <force_push_commit_sha>

Design Guardrails for Future Preventives:

Implement a pre-commit Git hook to reject destructive operations.
Set up automated tests and continuous integration pipelines.
Educate the team on best practices like:
MARKDOWN
- Always create feature branches for experiments.
- Use `git stash` or temporary branches for local development.
- Perform code reviews before merging changes into master.

#### Closing Summary

By carefully identifying the lost commits, using Git commands to recover them, and implementing guardrails, we can mitigate future risks. The key is to ensure that all team members are aware of best practices and have the necessary tools to prevent such issues.

#### Common Mistakes/Follow-Ups
Failing to use git reflog properly.
Not educating the team on preventive measures.
Overwriting sensitive data without proper backups or logs.

