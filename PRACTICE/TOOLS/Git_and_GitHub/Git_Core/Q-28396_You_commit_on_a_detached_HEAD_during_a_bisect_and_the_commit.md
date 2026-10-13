---
type: "practice"
difficulty: "Intermediate"
---

# You commit on a detached HEAD during a bisect and the commits look orphaned afterward. Explain what happened at the ref level, how to save the work into a branch, and when gc would eventually reclaim them.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Atomic Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Atomic_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you commit on a detached HEAD during git bisect, your commits can indeed appear orphaned afterward if they are not properly attached to an existing branch. Let's break down what happened, how to save the work into a branch, and when these commits might be reclaimed by Git.

What Happened
Detached HEAD: During a git bisect, you often check out a commit using its hash or tag, leaving you in a detached HEAD state. This means your working directory is checked out to a specific commit, but there's no branch name associated with it.
Committing Changes: When you make changes and commit them in this detached HEAD state, the commits are created without being linked to any existing branch.
How to Save the Work into a Branch
Identify the Commit: Use git reflog or git log --pretty=format:'%h' to find the hash of the commit where you made changes.
Create a New Branch: Switch back to your main branch (e.g., main) and create a new branch from that commit:
SH
git checkout -b my-new-branch <commit-hash>
Verify the Branch: Ensure that the new branch has all the necessary changes:
SH
git log --oneline
When Git Would Eventually Reclaim Them
Garbage Collection (gc): Git performs garbage collection to clean up unreferenced commits, but this process is not immediate. Commits are marked for garbage collection when they have no references.
Refcount: The refcount of a commit decreases as it gets removed from references like branches or tags. When the refcount reaches zero, these commits become eligible for garbage collection and deletion.

#### Common Mistakes
and Follow-Ups
Not Saving Changes Properly: Forgetting to create a branch after making changes in detached HEAD mode.
Overlooking git reflog: Not using reflog to find the exact commit hash before creating a new branch.
Confusion with git reset: Confusing reset commands which can move commits and potentially orphan them if not used carefully.
Summary

Understanding how Git manages branches and detached HEAD states is crucial, especially when you are performing detailed operations like bisecting. Always ensure that your changes are saved into a branch to avoid losing work. Be mindful of garbage collection and refcount mechanisms, as they can reclaim unreferenced commits.

**Code:**

```
SH
Copy
git checkout -b my-new-branch <commit-hash>
```

