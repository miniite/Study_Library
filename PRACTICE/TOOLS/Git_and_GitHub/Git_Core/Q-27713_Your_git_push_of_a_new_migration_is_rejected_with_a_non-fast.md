---
type: "practice"
difficulty: "Intermediate"
---

# Your git push of a new migration is rejected with a non-fast-forward error because a teammate pushed first. How do you fix it without losing either migration?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Rejected Pushes](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rejected_Pushes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering a "non-fast-forward" error while pushing a migration to Git, it means that someone else has committed changes to the branch in your absence. To resolve this issue and merge both migrations without losing any work, you need to rebase or manually incorporate the teammate's changes into your local repository before pushing again. Here’s how you can approach this:

Understand the Error: The non-fast-forward error occurs because Git cannot automatically fast-forward your branch to match the remote state. This means that there are newer commits on the remote branch that need to be incorporated.

Fetch and Rebase: First, fetch the latest changes from the remote repository:

BASH
git fetch origin

Then rebase your local branch onto the fetched updates:

BASH
git rebase origin/<branch-name>

Resolve Conflicts: If there are conflicts during the rebase process, Git will pause and allow you to resolve them manually. After resolving each conflict, continue with:

BASH
git add .
git rebase --continue

Push Again: Once all conflicts are resolved and your branch is rebased, push it again using git push origin <branch-name>.

By following these steps, you ensure that both migrations are integrated correctly without losing any changes from either migration.

**Code:**

```bash
# Fetch the latest remote changes
git fetch origin

# Rebase your local branch onto the fetched updates
git rebase origin/<branch-name>

# Resolve conflicts if they arise during the rebase process
git add .
git rebase --continue

# Push the rebased branch to the remote repository
git push origin <branch-name>
```

**Example input / output:**

```text
# Initial state
Local: A -> B (A and B are migration commits)
Remote: A -> C (C is a commit by your teammate)

# After fetching and rebasing
Local: A -> C' (rebased version of B on top of C)
```

**Flow:**

1. Yes
2. No
3. Fetch latest remote changes
4. Fetched repository
5. Rebase local branch ontofetched updates
6. Resolve conflicts?
7. Mark resolved files andcontinue rebase
8. Push rebased branch toremote
9. Continue rebase with'--continue'

