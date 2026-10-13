---
type: "practice"
difficulty: "Intermediate"
---

# A PR has 30 review comments on a stored-procedure change and its history is a mess of fix-review commits. Clean it up with interactive rebase and --autosquash while preserving reviewers' ability to see what changed since their last look.

**Difficulty:** Intermediate · **Levels:** SDE-1, SDE-1/2 · [↑ Interactive Rebase](<../../../../TOOLS/Git_and_GitHub/Git_Core/Interactive_Rebase.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To clean up a Pull Request (PR) with 30 review comments and a messy commit history, we'll use git rebase interactively with autosquash to maintain an organized and readable history while preserving context for reviewers. This process involves several steps:

Understand the Current State: First, ensure you understand the current state of your branch by running commands like git log --oneline or git reflog. Identify the commit where the PR starts.

Start Interactive Rebase: Use git rebase -i <start-commit> to start an interactive rebase from a specific commit. In this case, you might want to pick a commit that aligns well with the start of your PR.

Autosquash Commits: Utilize --autosquash in your rebasing process to automatically squash commits and preserve their messages. This will help reduce noise in the commit history.

Review and Edit: During the rebase, you’ll be prompted to edit commits as needed. This is where you can address comments, squish similar changes together, or split them if necessary.

Resolve Conflicts: If any conflicts arise during the rebase, resolve them manually before continuing with git add followed by git rebase --continue.

Push Updated History: Finally, push your updated history to GitHub using git push --force-with-lease. This will update your remote branch and ensure that reviewers can see the cleaned-up version of the code.

**Code:**

```
SH
Copy
# Start an interactive rebase from a specific commit
git rebase -i <commit-hash>

# Example command to autosquash commits interactively
git rebase --autosquash -i <start-commit>

# Resolve conflicts if any during the rebase process
# git add <resolved-files>
# git rebase --continue
```

**Flow:**

1. Yes
2. No
3. Start Interactive Rebase
4. Is there a specific commit?
5. start-commit
6. Run 'git rebase -i<commit-hash>' or use--autosquash
7. Review and Edit Commits
8. Resolve Conflicts (if any)
9. Use 'git add' to stage changes,then 'git rebase --continue'
10. Pull Updated History with 'gitpush --force-with-lease'

