---
type: "practice"
difficulty: "Deep"
---

# You need to split a giant already-merged 'add whole schema' commit into per-table logical commits for a clean future revert story. Walk through interactive rebase 'edit' plus reset, and the exact risks if the commit is already public and branched from.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Interactive Rebase](<../../../../TOOLS/Git_and_GitHub/Git_Core/Interactive_Rebase.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When you need to split a large merge commit into smaller logical commits using Git and GitHub, you can use interactive rebase with git rebase -i. This technique allows you to edit history, making it easier to break down complex changes into more manageable pieces. However, if the commit is already public and branched from, there are significant risks involved, primarily related to disrupting a shared workflow.

Let's walk through the process step by step:

#### Overview
We will use git rebase -i in interactive mode to split the merge commit into smaller commits for each table.

#### Assumptions
The commit is large and contains changes from adding an entire schema, including multiple tables.
Goal: Split this single commit into multiple commits, one per table.
Step-by-Step Solution

Identify the Commit:

SH
git log --oneline

This command will show you a list of recent commits, helping you identify the large merge commit that needs to be split.

Start Interactive Rebase:

SH
git rebase -i HEAD~N

Replace N with the number of commits back from your current head (e.g., if it's the most recent commit, use 1).

Edit Mode: When you run git rebase -i, Git will open an editor showing a list of recent commits. The commit that needs to be split should have "pick" in front of its SHA hash.

Replace with 'edit': Change the word pick to edit for the large merge commit, saving and closing the editor.

Review and Split: After starting the rebase process, Git will stop at the first commit marked as edit. Use git status to see which files have been modified.

Stage Changes:

SH
git add <file1> <file2>

Add changes for each table you want to create a separate commit for. You can use Git's staging area to selectively stage only the parts of the file that pertain to one table.

Commit: After staging, commit with a descriptive message.

SH
git commit --amend -m "Add <table1>"

Repeat: Continue this process for each table until all changes are split into separate commits.

Continue Rebase: Once you've created the desired number of smaller commits, continue the rebase with git rebase --continue.

Risks and Considerations

Public Branches: If the large commit is part of a public branch that has already been shared or merged into other branches, splitting it can cause issues:

Other developers might have pulled the large commit.
The history in Git will be rewritten, which could break their local clones if they haven't done a git pull recently.

Communication: Inform your team about the upcoming changes to ensure no one relies on the current state of the repository.

Test Impact: Ensure that splitting the commit does not affect any automated tests or build processes relying on the large commit.

#### Closing Summary
and Common Mistakes
Mistake 1: Overlooking communication with your team, leading to unexpected conflicts.
Mistake 2: Not testing thoroughly after changing history.
Mistake 3: Failing to consider how this change impacts other branches or repositories that have already pulled the large commit.

By following these steps carefully and understanding the risks involved, you can effectively split a large merge commit into smaller logical commits for better version control and maintainability.

