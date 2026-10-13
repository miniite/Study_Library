---
type: "practice"
difficulty: "Intermediate"
---

# How does Git decide whether a merge can be a fast-forward, and how can you force a non-fast-forward instead?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rejected Pushes](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rejected_Pushes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how Git decides if a merge is a fast-forward and how to force a non-fast-forward merge, let's break down the concepts first. A fast-forward merge in Git simply means that one branch can be moved forward by updating its pointer to match another branch’s latest commit. This happens when there are no divergent changes between the two branches. On the other hand, a non-fast-forward merge occurs when there are divergent changes and requires creating a new merge commit.

Git decides whether a merge is fast-forwardable based on the history of commits in both branches being merged. If the target branch's latest commit can be reached by traversing only forward from your current branch’s tip, then Git considers it a fast-forward operation. Otherwise, you need to create a new merge commit, resulting in a non-fast-forward merge.

Here are the steps to force a non-fast-forward merge:

Ensure You Have the Latest Changes: Before performing a non-fast-forward merge, make sure your local branch is up-to-date with the remote branch.
Check if Fast-Forward is Possible: Use git log or git cherry-pick to inspect the history and confirm that a fast-forward merge isn't possible due to divergent changes.
Initiate the Merge: Run git merge <branch> to start the merge process.
Force a Non-Fast-Forward Merge: If Git reports a conflict, resolve it by manually editing files or using interactive rebase.

Let’s dive deeper into these steps with an example and some practical insights:

#### Triage
What's Being Probed

The interviewer is likely probing your understanding of:

The basic principles of Git branches and history.
How Git tracks branch relationships.
The concept of fast-forward vs non-fast-forward merges.
Practical commands to handle merge operations.

#### First Principles Solution
Step 1: Ensure Your Branch Is Up-to-Date
BASH
git fetch origin
git checkout <your-feature-branch>
git pull origin <base-branch>

This ensures your feature branch is up-to-date with the latest changes from the base branch (e.g., main or master).

Step 2: Check for Fast-Forwardability

Run git merge-base <upstream-branch> <current-branch> to check if a fast-forward is possible. If it returns a commit ID, Git will attempt to do a fast-forward.

BASH
git merge-base main feature-branch

If no conflict arises and the command completes without error, you know that a fast-forward merge could be done.

Step 3: Force Non-Fast-Forward Merge

Since Git will automatically try to perform a fast-forward merge when possible, you can force it by using git merge --no-ff <branch>. This tells Git not to use the fast-forward option and creates a new merge commit instead.

BASH
git checkout feature-branch
git merge --no-ff main
Step 4: Resolve Conflicts (If Any)

If conflicts arise during the non-fast-forward merge, you need to resolve them manually:

Inspect Conflicts: Git will notify you of the files with conflicts.
Edit Files: Open conflicted files and manually edit them to resolve the conflicts.
Add Resolved Files: After resolving conflicts, stage the resolved files using git add <file>.
BASH
git add <resolved-file>
Step 5: Complete the Merge

Once all conflicts are resolved, complete the merge by committing:

BASH
git commit -m "Merge branch 'main' into feature-branch"

#### Closing Summary
and Common Mistakes

Fast-forward merges can save time when branches have not diverged. However, forcing a non-fast-forward merge is essential for documenting conflicts or showing the history of changes explicitly. Be cautious with forced commits; they can complicate history if not needed.

#### Common mistakes
include:

Not checking the fast-forwardability before attempting to force it.
Ignoring merge conflicts and proceeding without resolving them properly.
Committing prematurely, leading to a messy commit history.

#### Follow-Ups

The interviewer might ask about additional Git commands related to branching strategies or how merging affects remote branches. Be prepared to discuss best practices for maintaining branch integrity and managing divergent histories efficiently.

