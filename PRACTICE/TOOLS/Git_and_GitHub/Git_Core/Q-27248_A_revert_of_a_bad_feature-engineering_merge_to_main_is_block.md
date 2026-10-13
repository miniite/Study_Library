---
type: "practice"
difficulty: "Deep"
---

# A revert of a bad feature-engineering merge to main is blocking the later re-merge of the same feature because Git thinks it's already applied. Explain the mechanism at the DAG level and the exact sequence to cleanly re-introduce it in the training repo.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverting Merges](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverting_Merges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a situation where a revert blocks further merges due to perceived duplication, we need to understand how Git tracks changes and resolves conflicts at the DAG (Directed Acyclic Graph) level. Specifically, we'll delve into the underlying mechanisms of Git's merge process and propose a step-by-step solution to reintroduce the feature engineering.

The core issue here is that Git’s DAG keeps track of every commit as a node, with edges representing the parent-child relationships between commits. When you attempt to merge branches, Git tries to find the best common ancestor between them. If a revert has been applied, Git will see this change as having already occurred and might block further merges because it thinks the changes are redundant.

To resolve this, we need to manually re-introduce the feature engineering changes while ensuring Git understands the historical context correctly.

Identify the problematic commits: Determine which commit(s) were reverted that caused the merge conflict.
Revert the revert: Reapply the original change in a new commit.
Squash or rebase: Ensure the history is clean and merges smoothly.
Force push if necessary: If changes are pushed to main, force pushing might be required to overwrite the problematic commit.

Let's break this down step-by-step:

Step 1: Identify the Problematic Commits

First, we need to identify which commits were reverted and caused the merge conflict. Typically, Git logs will show a message indicating that a revert operation was performed on a specific commit. For example:

DIFF
git log --oneline

You might see something like:

CODE
abc123 (HEAD -> feature-engineering) Revert "Fix bug in feature X"
def456 Fix bug in feature X

Here, def456 is the original commit that was reverted.

Step 2: Revert the Revert

Next, we need to reapply the original change. We can do this using a new commit:

SH
git checkout feature-engineering
# Make sure you are on the correct branch.
git cherry-pick def456

Alternatively, if def456 is no longer in your history, you might need to recommit the changes manually by checking out the relevant files and committing them.

Step 3: Squash or Rebase

To ensure a clean merge, we might want to squash the new commit with its parent:

SH
git rebase -i HEAD~2

This will open an editor where you can mark one of the commits for squashing. After saving and closing the editor, Git will automatically combine the changes.

If the branch has multiple unrelated changes that should be kept separate, a simple git commit --amend might suffice to update the last commit message and include any additional files.

Step 4: Force Push if Necessary

If you have already pushed the changes to main, you may need to force push them:

SH
git push origin feature-engineering --force

Be cautious with force pushes as they can overwrite history for collaborators. Ensure all team members are aware and agree before proceeding.

#### Closing Summary
and Common Mistakes

To summarize, we identified the original commit that was reverted, reapplied it in a new commit, ensured the history was clean using rebase or squash, and force pushed if necessary to resolve the merge conflict.

#### Common mistakes
include not thoroughly understanding Git's DAG mechanism, performing force pushes without coordinating with team members, and failing to track down the exact commits involved. Always communicate clearly with your team during such operations to avoid confusion or data loss.

**Code:**

```
SH
Copy
git checkout feature-engineering
git cherry-pick def456
git rebase -i HEAD~2
git push origin feature-engineering --force
```

