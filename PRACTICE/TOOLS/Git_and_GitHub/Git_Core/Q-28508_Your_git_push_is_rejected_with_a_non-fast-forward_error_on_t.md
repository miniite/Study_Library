---
type: "practice"
difficulty: "Intermediate"
---

# Your git push is rejected with a non-fast-forward error on the deploy repo. What does it mean and how do you resolve it without losing work?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Rejected Pushes](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rejected_Pushes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you encounter a "non-fast-forward" error while trying to push to a Git repository, especially in a deployment context like GitHub, it means that the remote branch has been updated by someone else since the last time your local copy was synchronized with the remote branch. This can happen if other team members have pushed new commits or branches that conflict with yours.

#### Triage
Understanding the Problem and What's Being Probed

The question is fundamentally about resolving a Git conflict in a deployment scenario, ensuring you don't lose work, and understanding the underlying Git mechanics. The interviewer might be probing your knowledge of Git workflows, branching strategies, and best practices for handling merge conflicts.

#### Assumptions

You have experience with basic Git operations.
You are familiar with GitHub's workflow.
Step-by-Step Solution from First Principles
1. Understand the Context

In a deployment scenario, you might be working on features or bug fixes that need to go live in your production environment. When pushing changes to the main branch (or another protected branch), Git enforces policies such as non-fast-forward checks to ensure consistency and prevent accidental overwriting of recent commits.

2. Check the Current State

First, check the status of your current local repository:

SH
git status

This command will show you whether there are any uncommitted changes or if you need to stash them away for later. If you have uncommitted changes, commit them before proceeding.

3. Fetch Latest Changes from Remote

Use git fetch to get the latest updates from the remote repository:

SH
git fetch origin

This will update your local copy of the branch with the newest commits from GitHub without merging or pushing anything yet.

4. Rebase Your Work onto the Latest Remote Commits

Rebasing allows you to integrate changes from the upstream (in this case, origin/main) into your current branch, ensuring that your work is based on the latest code:

SH
git rebase origin/main
This step will replay all of your local commits on top of the latest remote commit.
If there are any conflicts, Git will pause and allow you to resolve them manually.
5. Resolve Conflicts (if any)

If rebase encounters a merge conflict, it will stop at the first conflict and give you an opportunity to resolve it. Common tools like git mergetool can be used to help with this process:

SH
git mergetool

Follow the prompts of your text editor or merge tool to complete the resolution.

6. Continue Rebase if Conflicts are Resolved

After resolving all conflicts, continue the rebase:

SH
git rebase --continue

Repeat this step until there are no more conflicts.

7. Push Your Changes

Once your work is rebased and you have resolved any conflicts, you can push it to the remote repository:

SH
git push origin main

#### Closing Summary

Handling a non-fast-forward error involves understanding Git's history and using tools like fetch and rebase to keep your local branch in sync with the remote. This process ensures that you don't lose any of your work while maintaining the integrity of the remote repository.

#### Common Mistakes/Follow-Ups
Not checking for uncommitted changes: Always ensure there are no uncommitted changes before pushing.
Failing to resolve conflicts correctly: Properly using merge tools and understanding how Git resolves conflicts can save time.
Pushing directly without rebasing: Forcing a push (git push --force) should be avoided as it can overwrite others' work.

Always use rebase for integrating changes from the upstream, and merge for merging branches if rebase is not suitable due to complexity or history issues.

