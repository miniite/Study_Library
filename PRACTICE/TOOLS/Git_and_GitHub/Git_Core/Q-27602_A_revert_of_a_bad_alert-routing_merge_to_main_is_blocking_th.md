---
type: "practice"
difficulty: "Deep"
---

# A revert of a bad alert-routing merge to main is blocking the later re-merge of the same correlation improvement because Git thinks it's already applied. Explain the DAG mechanism and the exact sequence to cleanly re-introduce the correlation change into your release train.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reverting Merges](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reverting_Merges.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with a situation where a revert of a bad alert-routing merge is blocking a subsequent re-merge in Git, understanding the Directed Acyclic Graph (DAG) mechanism and using a clean workflow to reintroduce changes becomes crucial. In this case, we need to ensure that the correlation improvement change can be cleanly reintroduced without being blocked by the reverted state.

#### Triage
Understanding the Problem

The core issue here is that Git considers the bad merge already applied because it was originally committed, and then reverted with a subsequent commit. This creates a conflict in the history, making re-merging difficult. The DAG mechanism in Git helps us understand the history of commits and their dependencies.

#### First Principles Solution
Introducing the Correlation Change

To cleanly reintroduce the correlation improvement change, we need to follow these steps:

Identify the Problem Commits: Locate the bad merge commit (let's call it B) and its revert (R). The DAG will show that R is a parent of B.

Clean Up the History:

Create a new branch from the most recent stable main or release branch.
Reapply the correlation improvement changes as if they were never reverted.

Rebase and Squash: Use Git rebase to simplify the commit history, ensuring that only necessary changes are applied.

Merge Safely: Merge the cleaned-up branch into your release train without introducing any unnecessary conflicts or state.

Step-by-Step Solution

Let's assume we have the following branches and commit history:

CODE
main -> A1 -> B (bad merge) -> R (revert of B) -> A2 -> ... (release train)
           \                       /
            -> C (correlation improvement change) <- clean branch origin
Step 1: Identify Problem Commits
B: Bad merge commit
R: Revert commit
Step 2: Clean Up the History

Create a new branch from main:

SH
git checkout -b clean-correlation

Reapply the correlation improvement change (C):

SH
git cherry-pick C

Ensure no other unnecessary changes are introduced by squashing or cleaning up unrelated commits:

SH
git rebase --interactive A1

After the interactive rebase, ensure only C and any necessary clean-up changes remain.

Step 3: Merge Safely
Rebase clean-correlation onto main to integrate the changes cleanly:
SH
git checkout main
git pull origin main --rebase
git checkout clean-correlation
git rebase main

If conflicts arise during the rebase, resolve them appropriately.

Once resolved, push the cleaned-up branch back to GitHub and merge it into your release train:

SH
git checkout main
git merge --no-ff clean-correlation

#### Closing Summary
+ Common Mistakes/Follow-Ups

By understanding the DAG mechanism, we can effectively manage complex commit histories in Git. The key steps are identifying the problematic commits, cleaning up the history, and merging safely without introducing conflicts.

#### Common Mistakes
Forgetting to identify all relevant commits before attempting a clean-up.
Not using rebase correctly, leading to unnecessary merge commits.
Overwriting or losing important changes during cleanup.

#### Follow-Ups

#### Cons
ider setting up Git hooks or scripts for automated cleanup of such issues in the future.
Document the process and share it with team members to prevent similar issues.

**Code:**

```
SH
Copy
git checkout -b clean-correlation
git cherry-pick C
git rebase --interactive A1
git checkout main
git pull origin main --rebase
git checkout clean-correlation
git rebase main
git checkout main
git merge --no-ff clean-correlation
```

**Flow:**

1. main
2. bad_merge
3. revert
4. A2
5. correlation_improvement
6. Clean-up history
7. Rebase and merge

