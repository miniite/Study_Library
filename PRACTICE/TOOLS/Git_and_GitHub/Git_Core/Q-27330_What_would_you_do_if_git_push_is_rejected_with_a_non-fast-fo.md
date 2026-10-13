---
type: "practice"
difficulty: "Intermediate"
---

# What would you do if git push is rejected with a non-fast-forward error while pushing prompt changes to a branch a teammate also pushed to?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Rejected Pushes](<../../../../TOOLS/Git_and_GitHub/Git_Core/Rejected_Pushes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering a non-fast-forward error during a Git push, it indicates that your local branch has diverged from the remote branch since you last pulled, and there are conflicting updates between your commits and those made by another team member. This situation can be managed by following these steps:

Understand the Error: The error message typically looks like this: "To prevent you from losing work, merge the remote changes into your working tree before pushing again." Essentially, Git is warning you that it's not safe to force push because doing so could overwrite other team members' commits.

Identify the Problem: The main issue here is the conflicting updates between branches. You need to resolve these conflicts by merging or rebase them in a controlled manner. This ensures that both sets of changes are preserved and that the history remains intact, making it easier for others to review and understand what has been done.

Fetch Remote Changes: Start by fetching all the latest changes from the remote repository:

BASH
git fetch origin

Merge or Rebase: You can either merge the remote branch into your local branch (to keep history linear) or rebase your commits onto the updated remote branch (which can be useful for cleaner histories):

Merging:

BASH
git merge origin/branch_name

This will integrate the changes from origin/branch_name into your current branch. Git will try to automatically resolve any conflicts.

Rebasing:

BASH
git rebase origin/branch_name

Rebasing takes your local commits and reapplies them on top of the updated remote branch, potentially resolving conflicts along the way.

Resolve Conflicts: If Git cannot automatically resolve all conflicts, it will pause the merge/rebase process to allow you to manually resolve any conflicts in the files where differences exist. Once resolved, continue with:

BASH
git add <resolved-files>
git rebase --continue  # For rebasing
git commit  # After merging if needed

Push Again: After resolving and integrating all changes, you can safely push your updated branch to the remote repository:

BASH
git push origin branch_name

Communication: It's a good practice to inform your teammates that there are pending merge/rebase operations so they don’t push their changes until conflicts are resolved.

