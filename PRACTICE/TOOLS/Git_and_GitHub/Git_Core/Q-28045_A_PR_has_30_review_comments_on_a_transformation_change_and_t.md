---
type: "practice"
difficulty: "Intermediate"
---

# A PR has 30 review comments on a transformation change and the history is a mess of 'fix review' commits. Clean it up with interactive rebase and --autosquash while preserving reviewers' ability to see what changed since their last look at the SQL.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Interactive Rebase](<../../../../TOOLS/Git_and_GitHub/Git_Core/Interactive_Rebase.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clean up the Git history in a pull request (PR) with 30 review comments, where the commit history is cluttered with 'fix review' commits, we can use interactive rebase and --autosquash. This process will help us maintain a cleaner commit history while preserving the reviewers' ability to see what has changed since their last look at the SQL. Let's walk through this step-by-step.

#### Triage
What We're Doing

The core of the question is about cleaning up a Git history that is messy due to numerous 'fix review' commits, ensuring we preserve the context of each change so reviewers can easily understand what has been altered since their last review. The key here is to use git rebase -i with --autosquash to combine and squash commits while maintaining a clear record of changes.

#### Assumptions
We are working on a Git repository where the branch in question is already created.
There are 30 commits, but only specific ones need reworking due to review comments.
The reviewers have already provided their feedback via comments.

#### First Principles Solution

Identify Commits:

Use git log or git reflog to identify the commit hashes of the relevant 'fix review' commits that we want to squash together.

Interactive Rebase:

Start an interactive rebase using git rebase -i <commit-hash>~n, where <commit-hash> is the hash of the commit before the first problematic commit, and n is the number of commits you wish to include in this rebase (e.g., 30 for all).
SH
git rebase -i HEAD~30

Mark Commits for Squashing:

In the interactive rebase editor, mark the 'fix review' commits with a s to squash them.
Ensure that reviewer comments are preserved by manually editing each commit message as needed.

Autosquash and Merge Commit Messages:

Use --autosquash to automatically combine the commit messages of the squashed commits, ensuring reviewers can still see what was changed since their last look at the SQL.

Commit Changes:

After making the necessary changes in the editor, save and close the file.
Resolve any conflicts if they arise during the rebase process.

Push Changes with Rebase:

Push the rebased branch back to the remote repository using git push --force, as interactive rebase will alter commit history.
Update the PR on GitHub or GitLab accordingly.

#### Closing Summary
and Common Mistakes/Follow-Ups

By following these steps, we can clean up the messy commit history while preserving the context of each change for reviewers. The key is to use git rebase -i judiciously and ensure that commit messages are clear enough so that reviewers understand the changes made.

#### Common Mistakes

Forgetting to mark commits with s in interactive rebase.
Not using --autosquash to maintain clear commit messages.
Failing to resolve conflicts during the rebase process, which can lead to a broken history.

#### Follow-Ups

Verify that all reviewers have seen and approved the updated changes before proceeding.
Use Git blame or history tools to double-check if necessary after resolving conflicts.

**Code:**

```
SH
Copy
git log  # View recent commits for context
git rebase -i HEAD~30  # Start interactive rebase on last 30 commits
# Edit the file in the editor, marking 'fix review' commits with 's'
git push --force  # Push changes to remote repository after resolving conflicts if any
```

**Example input / output:**

```text
$ git log
commit abcdef1234567890 (HEAD -> feature/transform-change)
Author: Qwen <qwen@example.com>
Date:   Mon Jan 1 00:00:00 2024 -0800

    Fix review comments for SQL transformations
```

**Flow:**

1. Start Interactive Rebase
2. Identify Commits
3. Edit Interactive rebase File
4. Squash Commits using 's' ineditor
5. Autosquash Commit Messages
6. Commit Changes
7. Publish Rebased Branch with'git push --force'

