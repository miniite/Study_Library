---
type: "practice"
difficulty: "Basic"
---

# What is the difference between tracked, untracked, and ignored files in Git?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Gitignore](<../../../../TOOLS/Git_and_GitHub/Git_Core/Gitignore.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Git repository, understanding the state of your files—whether they are tracked, untracked, or ignored—is fundamental to effective version control and project management. Tracked files are those that have been added to Git's index but not yet committed, while untracked files are new changes in the working directory not recognized by Git at all. Ignored files, on the other hand, are specified as not to be tracked by your .gitignore file, ensuring they are never included in commits.

#### Triage
The question is asking about three distinct states of files within a Git repository—tracked, untracked, and ignored—and what each state means for version control. It’s important to clarify the definitions to avoid common pitfalls like accidentally committing changes that should be ignored or not recognizing new code modifications.

#### First-principles solution
Let's start by defining each term:

Tracked Files: These are files that Git tracks, meaning you can commit and push these changes to a remote repository. You add these files to the index using git add. They have been committed at least once in the past or are currently being tracked.

Untracked Files: These are new files that haven't been added to Git's tracking mechanism yet. You create them in your working directory and can stage them for addition by running git add. Untracked files are not part of any previous commit, but they will be included in the next commit if staged.

Ignored Files: These files are specified in a .gitignore file to be ignored by Git. They are neither tracked nor untracked; instead, they are excluded from being added or committed. This is useful for managing files that should not clutter your repository, such as build artifacts, cache directories, or temporary files.

#### Closing summary
Understanding the difference between these three states helps you manage your project's version control effectively. Tracked and ignored files require careful handling to avoid accidentally committing sensitive data, while untracked files give you flexibility in tracking new code changes during development.

#### Common mistakes/follow-ups

Forgetting to add a file that should be tracked using git add.
Accidentally including important files in your .gitignore by mistake.
Confusing the state of ignored and untracked files, leading to issues with version control.

