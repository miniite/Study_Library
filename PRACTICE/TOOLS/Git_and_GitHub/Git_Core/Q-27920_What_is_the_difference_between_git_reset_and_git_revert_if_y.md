---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between git reset and git revert if you need to undo a bad transformation-model commit that is already pushed to a shared branch?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reset vs Revert](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reset_vs_Revert.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, understanding the differences between git reset and git revert can be crucial, especially when you need to correct a bad transformation model commit that has already been pushed to a shared branch. In this context, it's important to distinguish between modifying history (with git reset) versus creating a new commit that undoes the changes (with git revert). Let’s explore these commands in detail.

#### Triage
Understanding git reset and git revert
Git Reset: This command is used to undo changes, moving your working directory, staging area, or repository history. It can be used at different stages of the Git workflow.
Git Revert: This command creates a new commit that undoes all the changes introduced by another commit, effectively reversing its effects without altering the project’s commit history.
First Principles: The Core Solution
git reset

git reset is primarily used to undo commits and changes before they are committed. There are three main types of reset:

Hard Reset: Moves the branch pointer to a previous commit, discarding uncommitted changes in your working directory.
Mixed Reset: Similar to hard but also updates the index (staging area), which means you can still get back to the state before using git checkout.
Soft Reset: Only moves the branch pointer and does not change the index or the working directory.
git revert

git revert is used to undo a commit by creating a new one that undoes the changes of the original commit. This is particularly useful when you have already pushed a commit, as it preserves your history and allows for a clean rollback process.

#### Example Scenario

Suppose you have made a bad transformation model commit A, which has been pushed to the shared branch. You want to correct this mistake without altering the project’s historical commits.

Using git revert

To correct the issue using git revert:

First, identify the commit hash of the bad commit (A).
Run the command:
SH
git revert <commit-hash>
This will create a new commit that undoes the changes introduced by the original commit.
Using git reset

To correct the issue using git reset, you would typically do:

If the bad commit is still in your local branch, use:
SH
git reset --hard <commit-hash-before-A>
This will move the branch pointer to a previous commit and discard changes from the bad commit.
If the bad commit has already been pushed, using reset might not be ideal since it would alter history.

#### Closing Summary
+ Common Mistakes
Common Mistake: Using git reset --hard on a remote commit can cause issues with collaborators who have pulled that commit and may need to manually revert their changes.
Best Practice: Use git revert when the bad commit has been pushed, as it maintains your project’s history and allows for better collaboration.

#### Example
Command

Suppose you want to revert the last bad transformation model commit:

SH
# Identify the commit hash of the bad commit (A)
$ git log

# Revert the commit using the commit hash
$ git revert <commit-hash-of-A>

By following these steps, you can ensure that your Git workflow remains clean and collaborative.

