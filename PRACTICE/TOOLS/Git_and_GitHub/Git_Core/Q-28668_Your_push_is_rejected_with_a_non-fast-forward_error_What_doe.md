---
type: "practice"
difficulty: "Intermediate"
---

# Your push is rejected with a 'non-fast-forward' error. What does it mean and how do you fix it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rejected Pushes](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rejected_Pushes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you encounter a 'non-fast-forward' error while pushing to Git, it means that your local branch has commits that are not directly connected to the remote branch's latest commit. This can happen due to someone else committing changes to the same branch or if you have made multiple pushes to different points in history. To resolve this issue, you need to fetch the latest updates from the remote repository and rebase your local changes onto it.

Triaging the Problem

The 'non-fast-forward' error occurs because Git cannot directly fast-forward the remote branch with your local commits without rewriting history. This can lead to conflicts or loss of commit data if not handled correctly.

#### Assumptions

You are working on a team where multiple developers are contributing to the same branch.
There is no merge conflict between the branches.
You have push access to the remote repository.

#### Solution
from First Principles
Fetch Latest Changes: Use git fetch to get the latest updates from the remote repository without merging them into your local branch.
Rebase Local Branch: Reapply your local commits on top of the updated remote branch using git rebase origin/<branch_name>.
Force Push (if necessary): If you have no conflicts and are sure about overwriting the remote branch, you can use git push --force to force push your changes.
Step-by-Step Solution
Open a terminal or Git Bash.
Run git fetch. This will update your local repository with the latest changes from the remote repository without merging them.
Check the status of your branch using git status.
If you see any differences, rebase your local commits onto the updated remote branch by running:
BASH
git rebase origin/<branch_name>
Resolve any conflicts that arise during the rebase process.
After resolving conflicts, continue the rebase with:
BASH
git rebase --continue
Once the rebase is complete, force push your changes to overwrite the remote branch if necessary:
BASH
git push --force origin <branch_name>

#### Closing Summary
and Common Mistakes/Follow-Ups

Always be cautious when using --force as it can potentially overwrite others' work. It's generally better practice to use git pull, which fetches the latest changes and merges them into your local branch, creating a new commit on top of the remote one.

#### Common mistakes

Forgetting to fetch before rebasing.
Ignoring merge conflicts when resolving during rebase.
Using --force without understanding its implications.

#### Follow-ups
might include discussing scenarios where merging is preferable over rebasing and how to handle conflicts effectively.

