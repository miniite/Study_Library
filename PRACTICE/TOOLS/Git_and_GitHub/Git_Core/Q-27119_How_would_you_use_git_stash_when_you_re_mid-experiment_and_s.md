---
type: "practice"
difficulty: "Basic"
---

# How would you use git stash when you're mid-experiment and suddenly need to switch to a colleague's urgent data fix?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Git Stash](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Stash.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working on an experiment, it’s common to get temporarily sidetracked by urgent tasks that require immediate attention. Git stash is a powerful tool in such scenarios, allowing you to save your work without committing it or switching branches. Here's how you can use git stash in this situation:

Firstly, let's assume you're working on a feature branch called feature/new_model. You've made some significant changes but haven't yet committed them because the model isn't ready for testing just yet. Suddenly, your colleague informs you that there’s an urgent fix needed in the master branch to prevent data corruption.

To use git stash, follow these steps:

Stage Your Changes: Before stashing, ensure all changes are staged and added to the index if necessary.
Stash Your Work: Use git stash save "Message" to store your current work without committing it.
Switch Branches: Use git checkout master to switch to the colleague's branch where the urgent fix is required.
Apply Stashed Changes Later: Once you’ve completed the urgent task, return to your original feature branch using git checkout feature/new_model, and apply the stashed changes with git stash pop.

This process ensures that your experimental work remains intact but doesn't interfere with the immediate urgent requirements.

**Comparison:**

```
Command	Action
git add .	Stage all current changes.
git stash save "Mid-experiment changes"	Save the staged changes without committing them to a new stash entry.
git checkout master	Switch to another branch (in this case, master).
git commit -m "Urgent fix for data corruption"	Commit urgent fixes on the master branch.
git checkout feature/new_model	Return to your original experiment branch.
git stash pop	Apply stashed changes from previous branch back into current one, ready for further experimentation.
```

**Example input / output:**

```

Initial setup:

In the feature/new_model branch
Changes made but not committed

Urgent task scenario:

Run: git add .
Run: git stash save "Mid-experiment changes"
Run: git checkout master
Complete urgent fix and commit
Back to original branch: git checkout feature/new_model
Reapply stashed changes: git stash pop

Output:

CODE
Copy
$ git add .
$ git stash save "Mid-experiment changes"
Saved working directory and index state On feature/new_model: ...
$ git checkout master
Switched to branch 'master'
$ git commit -m "Urgent fix for data corruption"
[master 1234567] Urgent fix for data corruption
1 file changed, 2 insertions(+), 1 deletion(-)
$ git checkout feature/new_model
Switched to branch 'feature/new_model'
$ git stash pop
On branch feature/new_model
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)

        modified:   model.py

no changes added to commit (use "git add" and/or "git commit -a")
```

**Flow:**

1. Yes
2. Stash Work
3. Is work staged?
4. git add .
5. D
6. Switch to `master` branch
7. Complete urgent fix andcommit
8. Switch back to original featurebranch
9. Apply stashed changes with gitstash pop

