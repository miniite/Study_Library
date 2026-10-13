---
type: "practice"
difficulty: "Intermediate"
---

# What is git stash and give a scenario where it is useful.

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Git Stash](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Stash.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Git stash is a powerful tool in Git that allows you to temporarily save your changes without committing them or switching branches. It's particularly useful when you need to quickly switch to another task, such as fixing a bug or updating dependencies, and want to avoid making incomplete or unfinished changes part of the commit history.

#### Triage
The question is asking for an explanation of what git stash does and provides a practical scenario where it can be effectively used. The interviewer might probe your understanding of Git workflows and how you manage temporary changes while switching tasks. I'll assume that we are working in a collaborative environment where multiple features or bugs are being developed concurrently.

#### First-Principles Solution
Imagine you're a backend SDE-2/3 on a team responsible for developing new features, such as user authentication and payment processing, alongside fixing critical bugs. You've been working on the user-auth branch to add login functionality, but suddenly, the product manager asks you to fix a high-priority bug in the payment-processing feature that's causing revenue loss. Here’s how git stash can help:

Identify Current Work: Ensure you're aware of all uncommitted changes on your current branch (e.g., user-auth). These could be new files, modified code, or partial commits.
Stash Uncommitted Changes:
BASH
git stash
Switch Branches:
BASH
git checkout payment-processing
Fix the Bug: Work on the bug fix until it's resolved and tested thoroughly.
Return to Original Branch:
BASH
git checkout user-auth
Apply Stashed Changes:
BASH
git stash pop

#### Closing Summary
+ Common Mistakes/Follow-Ups: git stash is a valuable tool for managing temporary changes, especially in fast-paced development environments where tasks can change suddenly. It helps maintain a clean and organized commit history by allowing you to save incomplete work without committing it or switching branches.

#### Common mistakes
include:

Forgetting that git stash pop applies the changes back to your current branch.
Overusing stashes for long-term storage of features, which can complicate Git history management.

#### Follow-ups
might include discussions on other Git commands like cherry-pick, rebase, and strategies for managing multiple feature branches efficiently.

**Code:**

```bash
# Stashing changes
git stash

# Switching to another branch
git checkout payment-processing

# Fixing the bug
# (work done here)

# Returning to original branch and applying stashed changes
git checkout user-auth
git stash pop
```

**Flow:**

1. Identify current work
2. Stash uncommitted changes
3. Switch branches
4. Fix the bug
5. Return to original branch
6. Apply stashed changes

