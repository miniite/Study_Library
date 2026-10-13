---
type: "practice"
difficulty: "Intermediate"
---

# What would you do if git push is rejected with a non-fast-forward error while pushing an alert-rule change a teammate also pushed to?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Rejected Pushes](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rejected_Pushes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering a "non-fast-forward" error during a Git git push, it means that the branch you are trying to push has diverged from its remote counterpart, and Git cannot automatically update the remote because of unmerged changes. This situation often occurs when two people make conflicting changes to the same file or line in a repository. Here’s how I would handle this scenario as an AIOps engineer with intermediate experience.

To resolve this issue, we need to understand what happened on both local and remote branches, merge those changes appropriately, and then push again. The steps are:

Identify the Conflicts: Understand which parts of the file or code have conflicts.
Pull Latest Changes: Update your local branch with the latest remote changes.
Resolve Merges: Manually resolve any merge conflicts that arise due to the conflicting changes.
Stage and Commit Resolved Files: After resolving, stage the resolved files and commit them.
Push Again: Push the updated local branch back to the remote.

Let's break down each step in detail:

#### Triage

The question is asking how to handle a specific Git error when pushing changes that conflict with another teammate’s push. We need to ensure our local environment is up-to-date, resolve any merge conflicts, and then re-push the updated changes.

Step-by-Step Solution

Identify the Conflicts:

Run git status to understand what files have conflicts.
Look at the conflict markers in the file, which are typically indicated by lines like <<<<<<< HEAD, =======, and >>>>>>> <commit message>.

Pull Latest Changes:

Use git pull origin <branch-name> to fetch the latest changes from the remote branch.
This will merge the remote changes into your local branch, potentially creating conflicts that you need to resolve.

Resolve Merges:

Open the files with conflicts and manually edit them to integrate both sets of changes.
Ensure the code is coherent and follows the project’s coding standards.

Stage and Commit Resolved Files:

Stage the resolved files using git add <file-name>.
Commit the changes with a descriptive message, e.g., git commit -m "Resolved merge conflicts after syncing with teammate's push".

Push Again:

Finally, try pushing your updated branch again with git push origin <branch-name>.

#### Closing Summary

By following these steps, we can resolve the non-fast-forward error and ensure that our local changes are correctly reflected in the remote repository. This process helps maintain a clean and consistent codebase.

#### Common Mistakes/Follow-Ups
Failing to Pull Latest Changes: If you skip pulling latest changes before resolving conflicts, it might lead to further issues.
Ignoring Merge Conflicts: Not addressing merge conflicts properly can result in broken or untested code.
Poor Commit Messages: Descriptive commit messages are crucial for future reference and debugging.

**Example input / output:**

```text
$ git status
On branch feature-xyz
You are currently merging in 'origin/feature-xyz'.
Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)

    modified:   alert-rules/alert-rule.js

no changes added to commit (use "git add" and/or "git commit -a")

$ git pull origin feature-xyz
From https://github.com/example-repo
   53e2c7b..daa01f4  feature-xyz -> origin/feature-xyz
Auto-merging alert-rules/alert-rule.js
CONFLICT (content): Merge conflict in alert-rules/alert-rule.js
Automatic merge failed; fix conflicts and then commit the result.
```

