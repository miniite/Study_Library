---
type: "practice"
difficulty: "Intermediate"
---

# How would you use git revert to cleanly back out a bad alert-threshold change on a shared branch without rewriting history?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reset vs Revert](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reset_vs_Revert.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, git revert is an excellent tool for cleaning up changes in a way that doesn't alter the commit history. To back out a bad alert-threshold change on a shared branch without rewriting history, we need to identify the problematic commit, apply git revert, and ensure our workflow remains clean.

#### Triage

The question asks about using git revert to undo an unwanted change in a shared branch while preserving the original history. We'll assume that:

The change was made on a shared branch.
There are collaborators who may have based their work on this branch.
The branch is part of an AIOps (Artificial Intelligence for Operations) project, where alert thresholds might be critical.

#### First Principles Solution
Identify the Commit: Find the commit hash of the change that introduced the problematic threshold update.
Create a Revert Commit: Use git revert <commit-hash> to create a new commit that undoes the changes made by the original commit.
Push Changes: Push the revert commit to the shared branch, ensuring collaborators are aware of the change.
Detailed Steps

Identify the Problematic Commit:

Check recent commits on the shared branch using git log or git reflog.
Identify the specific commit that introduced the incorrect threshold (e.g., 9b3456789).

Create a Revert Commit:

BASH
git revert 9b3456789

This command will open an editor with a message suggesting to undo the changes introduced by commit 9b3456789. Make any necessary adjustments, save, and exit.

Push Changes:

BASH
git push origin <branch-name>

#### Closing Summary

Using git revert ensures that you back out a change without altering the original history of the branch, making it easier for others to follow and review the changes. This method is particularly useful in shared branches where multiple developers might be working concurrently.

#### Common Mistakes/Follow-Ups
Not Using Revert: Failing to use git revert can lead to merge conflicts or incorrect states if other commits are based on the problematic commit.
Overwriting History: Rewriting history with git rebase -i, for example, would change the project's commit history and could cause issues with collaborators who have already pulled the original branch.

Always communicate changes clearly to your team and ensure everyone is aware of any updates that might affect their work.

**Code:**

```bash
# Example commands
git revert 9b3456789
git push origin feature-branch
```

