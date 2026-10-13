---
type: "practice"
difficulty: "Intermediate"
---

# What does the reflog look like after a rebase, and how would you use it to get back to your pre-rebase state?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reset vs Revert](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reset_vs_Revert.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, understanding what the reflog looks like after a rebase is crucial because it helps in recovering from mistakes made during rebasing operations. Let's dive into how reflogs work, what they look like post-rebase, and how to use them effectively.

TL;DR

The reflog records all changes that happen to the tip of your branch. After a rebase, you can use the reflog to identify specific states before or after the rebasing process, allowing you to revert back to any previous state if needed.

#### Triaging the Question

This question is probing your understanding of Git’s internal mechanisms and how they handle branch histories during operations like rebase. The key concepts include:

Reflog: A record of all changes made to a reference (like HEAD or branches) in a repository.
Rebase: A process that moves the branch head to a new base commit, often used for cleaning up commit history.

#### Assumptions
You are familiar with basic Git commands and operations like branching, committing, and rebasing.
The rebase was done on a branch named feature from an initial state where it had commits A-B-C-D.
You want to revert the feature branch back to its pre-rebase state (commit C).

#### First-Principles Solution
Step 1: Understanding Reflog

The reflog is a log of actions that changed the tip of your branch, such as commits, resets, and rebases. It can be accessed using the command:

BASH
git reflog feature

This will show you all the operations performed on the feature branch.

Step 2: Analyzing Reflog Post-Rebase

After performing a rebase of the feature branch, the reflog might look something like this:

CODE
a9b7c63 (HEAD -> feature) : rebase 1/5: merge commit
8c0d1e4 : rebase 2/5: commit D on feature
5f6b9e2 : rebase 3/5: commit C on feature
1234567 : rebase 4/5: commit B on feature
2345678 (origin/feature) : initial state of feature branch

Here, a9b7c63 is the current tip of your branch after rebasing. The previous states are recorded with their respective SHA-1 hashes and descriptions.

Step 3: Reverting to Pre-Rebase State

To get back to the pre-rebase state (commit C), you can use the git reset command along with the appropriate reflog entry:

BASH
git reset --hard 5f6b9e2

This command will move your branch pointer and working directory back to commit 5f6b9e2, effectively undoing all changes made during the rebase.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, the reflog provides a way to track and recover from operations like rebasing. Understanding how it works is essential for managing your Git history effectively. Common mistakes include not regularly checking the reflog or forgetting that it only tracks changes on the branch tip.

#### Follow-Up
Questions
Can you demonstrate this with sample commands?
What are other uses of reflog in Git operations besides rebasing?
How does rebase compare to merging, and when would you use one over the other?

**Code:**

```bash
git reflog feature
git reset --hard 5f6b9e2
```

**Example input / output:**

```text
$ git reflog feature
a9b7c63 (HEAD -> feature) : rebase 1/5: merge commit
8c0d1e4 : rebase 2/5: commit D on feature
5f6b9e2 : rebase 3/5: commit C on feature
1234567 : rebase 4/5: commit B on feature
2345678 (origin/feature) : initial state of feature branch

$ git reset --hard 5f6b9e2
HEAD is now at 5f6b9e2 [commit C message]
```

