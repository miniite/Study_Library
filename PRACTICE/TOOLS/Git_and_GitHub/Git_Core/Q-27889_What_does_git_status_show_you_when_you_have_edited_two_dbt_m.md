---
type: "practice"
difficulty: "Basic"
---

# What does git status show you when you have edited two dbt model files but staged only one of them?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reviewing Changes with Diff](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reviewing_Changes_with_Diff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, git status provides a snapshot of your local repository's state relative to the remote repository or to a previous commit. In this specific scenario, if you have edited two dbt model files but staged only one of them, running git status will reveal which changes are uncommitted and which ones are staged for commit.

#### Triage

The question is asking us to understand what information git status would display in a situation where two files were edited, but only one was staged. This requires an understanding of Git’s staging area (also known as the index) and how git status reports the state of your working directory.

#### First-Principles Solution
Unstaged Changes: These are the changes that have been made to files in your working directory but haven't been added to the staging area yet.
Staged Changes: These are the changes that have been added to the staging area and are ready to be committed.

Given these definitions, when you run git status:

It will show a list of files with unstaged changes (i.e., files edited but not staged).
If a file is staged, it will also indicate which part of the file has been staged.

#### Example
Output

When you have two dbt model files (model_a.sql and model_b.sql) edited:

BASH
$ git status
On branch main
Your branch is up to date with 'origin/main'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   models/model_a.sql
        modified:   models/model_b.sql

no changes added to commit (use "git add" and/or "git commit -a")

In this output:

models/model_a.sql is shown as modified, indicating that it has changes not staged for commit.
models/model_b.sql is also shown as modified, but the fact that only one file was staged means we expect to see a specific message or behavior differentiating which of these files are staged.

#### Closing Summary
+ Common Mistakes/Follow-Ups

When running git status, unstaged changes will be listed along with their names. For staged changes, Git typically highlights the files and possibly marks parts of the file (e.g., lines) that have been added to the staging area. However, if only one file is staged, it might show a message like "Changes not staged for commit" without further detail unless you run commands like git diff --staged or check the status in more detail.

#### Common mistakes
include:

Assuming all unstaged changes are ready for commit.
Not understanding the difference between git add <file> and staging individual hunks (changes) within a file.

#### Follow-up
questions might include asking about how to stage specific parts of a file, what happens when you run git status after committing staged changes, or how to use Git's interactive staging feature (git add -p).

**Code:**

```bash
# Example git commands
$ git status
```

