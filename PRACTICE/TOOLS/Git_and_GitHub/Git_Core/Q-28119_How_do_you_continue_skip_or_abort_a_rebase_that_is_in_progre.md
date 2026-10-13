---
type: "practice"
difficulty: "Basic"
---

# How do you continue, skip, or abort a rebase that is in progress?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Rerere and Repeated Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rerere_and_Repeated_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working on Git and GitHub, it's common to encounter situations where you need to manage rebases actively. A rebase can sometimes get into a problematic state due to conflicts or other issues. Knowing how to continue, skip, or abort such a rebase is crucial for maintaining the integrity of your code repository.

#### Triaging the Question

The question essentially asks about managing an ongoing Git rebase. Specifically:

Continue: How do you resolve any merge conflicts and proceed with the rebase?
Skip: How can you bypass a specific commit during a rebase?
Abort: What steps are involved in stopping a rebase entirely?

#### Assumptions
You are familiar with basic Git commands.
A rebase is currently in progress, involving multiple commits.

#### First Principles Solution
Continuing a Rebase

To continue a rebase after resolving conflicts:

BASH
git rebase --continue

This command instructs Git to resume the rebase process after you've addressed any merge conflicts.

Skipping a Commit During a Rebase

If you want to skip a specific commit during a rebase, use:

BASH
git rebase --skip

This command tells Git to move on to the next step in the rebase process without applying the current commit.

Aborting a Rebase

To stop a rebase entirely and revert to your original branch state:

BASH
git rebase --abort

This command discards any changes made during the rebase and restores the repository to its pre-rebase state.

#### Closing Summary
+ Common Mistakes/Follow-Ups
Continuing: Ensure all conflicts are resolved before running git rebase --continue.
Skipping: Use --skip carefully, as it can lead to commit history that is difficult to understand.
Aborting: --abort is the safest way to stop a rebase; use with caution if you've made changes that might be needed later.

#### Common mistakes
include not properly resolving conflicts or skipping crucial commits. Always review and test your code after rebasing to ensure no issues are introduced.

Depth + Language

This solution covers the basic commands required for managing a Git rebase in a straightforward manner, suitable for a mid-level backend engineer who is familiar with Git but might be new to complex rebase scenarios.

**Code:**

```bash
# Continuing after resolving conflicts
git rebase --continue

# Skipping a specific commit
git rebase --skip

# Aborting the entire rebase
git rebase --abort
```

