---
type: "practice"
difficulty: "Deep"
---

# What is an interactive rebase and what can you do with it?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Interactive Rebase](<../../../../TOOLS/Git_and_GitHub/Git_Core/Interactive_Rebase.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Interactive rebase in Git is a powerful feature that allows you to modify the commit history of your branch interactively. This means you can squash, reorder, or even delete commits before they are pushed to the remote repository. This tool is particularly useful for cleaning up your local branch's history and ensuring it follows a logical flow.

#### Triaging the Question

The question asks about interactive rebase, which primarily tests your understanding of Git workflow and version control practices. The interviewer may be probing your knowledge on how to manage commit history effectively, especially in scenarios where you need to clean up or adjust commits before merging back into main branches.

#### Assumptions
and First Principles
You have a basic understanding of Git's git rebase command.
You are familiar with the concept of linear vs. non-linear commit histories.
Detailed Solution

Interactive rebase is performed using the --interactive option or by simply typing git rebase -i. Here’s how you can use it:

Preparation: Start by fetching the latest changes from the remote repository:

BASH
git fetch origin main

Initiate Rebase:

BASH
git checkout feature-branch
git rebase -i HEAD~N  # N is the number of commits you want to rebase.

This command opens an editor with a list of recent commits.

Interactive Editor: In the editor, you can see a list of your most recent commits:

CODE
pick 1234567 Commit message
pick 2345678 Another commit message

Reordering and Editing:

To reorder: Change pick to reword, edit, or squash followed by the commit hash.
To squash commits, change one of them from pick to squash (or s) and reword appropriately.

Finalizing Changes: Save and close the editor. Git will then ask you for a new commit message if you squashed or reworded any commits. Once done, continue rebasing with:

BASH
git rebase --continue

Resolve Conflicts: If there are conflicts during the rebase, resolve them and continue the process.

Pushing Changes: After successfully rebasing, push your changes to the remote repository using git push or git push -f if you need to force-push due to significant history changes:

BASH
git push origin feature-branch --force  # Use with caution!

#### Closing Summary
and Common Mistakes

Interactive rebase is a powerful tool for cleaning up your commit history, but it should be used judiciously. Common mistakes include:

Forgetting to save the editor after making changes.
Incorrectly forcing push (git push -f), which can overwrite other developers' work if not communicated properly.

#### Follow-Ups

The interviewer might ask about scenarios where you would use interactive rebase, or how it differs from git merge. Understanding these nuances will demonstrate your deep understanding of Git and version control practices.

**Code:**

```bash
git fetch origin main
git checkout feature-branch
git rebase -i HEAD~N
```

**Flow:**

1. Fetch latest changes
2. Fetch origin/main
3. Checkout feature branch
4. Initiate interactive rebase
5. [Pick, Reorder, Edit, Squash]
6. Save and close editor
7. Continue rebasing
8. Resolve conflicts if any
9. Push changes (force ifnecessary)

