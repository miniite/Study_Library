---
type: "practice"
difficulty: "Basic"
---

# How would you use git stash when you are mid-edit on a complex aggregation query but suddenly need to switch branches to review a teammate's ingestion fix?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git Stash](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Stash.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, git stash is an incredibly useful command that helps manage changes in your current work without committing them, allowing you to switch branches or perform other operations. In this scenario, you're mid-edit on a complex aggregation query but suddenly need to review a teammate's ingestion fix by switching branches.

First, let’s break down the situation and what git stash can do for us:

Current State: You are actively editing a file (e.g., an SQL script) in your current branch.
Immediate Need: You need to switch to another branch to review a teammate's changes.
Desired Outcome: Ensure you can return to the exact state of your aggregation query when you're done with the review.
Using git stash Step-by-Step

Check Your Current Status:

BASH
git status

This will show which files are modified and if they need committing.

Stash Your Changes: Use git stash to save your current changes in a stack, making them temporarily invisible.

BASH
git stash

If you have multiple files or the state is complex, it might be helpful to explain that Git will create a stash of all modified files.

Switch Branches: Now, switch branches using git checkout (or git switch if you're on a newer version of Git).

BASH
git checkout <target-branch>

Review the Changes: Once in the target branch, you can now review and test your teammate's ingestion fix.

Return to Your Original Branch: After reviewing, switch back to your original branch.

BASH
git checkout <original-branch>

Apply Stashed Changes: Apply the stashed changes using git stash apply or git stash pop.

BASH
git stash apply stash@{0}

If you want to keep these changes, use git stash pop which applies and keeps the changes.

#### Closing Summary
and Common Mistakes

By using git stash, you effectively save your work in a temporary location without committing it. This allows you to switch branches seamlessly for reviewing others' work or addressing urgent tasks.

#### Common Mistakes

Overwriting Stashes: If you stashed multiple times, remember that each stash is numbered (e.g., stash@{0}), and it’s easy to accidentally apply the wrong one.
Not Reverting Changes: After completing your review, be sure to revert any stashed changes if they’re not needed. Use git stash drop to remove them permanently.

In summary, git stash is a powerful tool for handling temporary work in Git repositories, especially when you need to quickly switch branches and come back later.

