---
type: "practice"
difficulty: "Basic"
---

# What happens to your uncommitted changes when you switch branches, and when will Git refuse to switch?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reset vs Revert](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reset_vs_Revert.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git, switching branches can have different outcomes depending on whether you have uncommitted changes or not. Here’s a quick overview:

TL;DR: Uncommitted changes affect your ability to switch branches. If there are uncommitted changes in the current branch, Git will generally refuse to switch until those changes are either committed or discarded.
What the question is really asking and what the interviewer is probing: The core of this question revolves around understanding Git's behavior when branching operations are performed. Specifically, it tests your knowledge on how Git handles uncommitted changes during a branch switch.

Let’s delve into the details:

Uncommitted Changes:

Staging Area (Index): Any files that have been modified and added to the staging area but not yet committed will be preserved when switching branches.
Working Directory: Files in your working directory that haven’t been staged or committed can cause Git to prevent a branch switch if they are modified.

Refusing to Switch:

Error Message: If you attempt to switch branches and have uncommitted changes, Git will typically respond with an error message like "You have modified [files] which would be overwritten by checkout."
Resolving the Issue:
Commit your current work before switching. Use git add . to stage all changes, then commit them using git commit -m "Your commit message".
Alternatively, you can discard uncommitted changes using git checkout -- .. This will revert local modifications but keep tracked files.

Why Git behaves this way: Git enforces a clean working state to ensure that branches remain consistent and stable. A dirty state (uncommitted changes) can lead to conflicts or data loss if not handled properly.

Now, let’s break down the first principles:

Understanding Branches:
Each branch in Git is essentially a pointer to a specific commit.
Current State of Files:
Any modifications that haven’t been staged or committed are part of your working directory and can cause conflicts if Git switches branches.

In practice, here’s what happens when you try to switch branches with uncommitted changes:

BASH
$ git status  # This will show any modified files
$ git checkout <new-branch>  # Attempting this will fail if there are uncommitted changes

If you get an error message indicating that uncommitted changes exist, you have a few options:

Commit the Changes:

git add . to stage all changes.
git commit -m "Your commit message" to finalize them.

Discard Changes:

git checkout -- <file> to discard specific file modifications.
git reset HEAD <file> to un-stage a file but keep the local modifications.

Stash Your Changes:

git stash temporarily saves your changes, allowing you to switch branches and then reapply them later with git stash pop.

Here’s an example of using stashing:

BASH
$ git status  # Show uncommitted changes
$ git stash    # Stash the changes for later use
$ git checkout <new-branch>  # Now should be able to switch without issues

Finally, let’s address some common mistakes and follow-ups:

#### Common Mistakes

Ignoring Git’s warning about uncommitted changes.
Assuming that Git will automatically handle branch switching with dirty files.

#### Follow-Ups

What happens if you have untracked files?
How can you ensure a clean working directory before branching?

By understanding these concepts, you can effectively manage your workflow and avoid common pitfalls in version control.

