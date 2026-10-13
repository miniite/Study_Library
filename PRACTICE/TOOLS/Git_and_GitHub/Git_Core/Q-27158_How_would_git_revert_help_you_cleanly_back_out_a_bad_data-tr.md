---
type: "practice"
difficulty: "Intermediate"
---

# How would git revert help you cleanly back out a bad data-transformation commit on a shared branch without rewriting history?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Reset vs Revert](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reset_vs_Revert.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To cleanly back out a bad data-transformation commit on a shared branch without rewriting history, Git revert provides an elegant and efficient solution. Essentially, it allows you to undo changes made in specific commits while maintaining their commit history. This is particularly useful when working with others who might be dependent on the original state of your repository.

#### Triage
Understanding the Question

The core question here is about using Git revert to address a bad data-transformation commit without altering the existing commit history. The interviewer is probing your understanding of Git workflows, especially in collaborative environments where shared branches are used. Additionally, they might be interested in how you handle potential conflicts and ensure code stability.

Key Assumptions
You have already pushed the problematic commit to a shared branch.
Other team members may depend on the current state of the repository or parts of the commit content.
Step-by-Step Solution from First Principles
1. Identify the Bad Commit

First, identify the exact commit hash that introduced the data transformation issue. You can use git log to find it:

SH
git log --oneline
2. Use git revert

To revert this specific commit without altering its history, you would use the git revert command followed by the commit hash or range of commits if multiple are involved:

SH
git revert -m 1 <commit-hash>

Here, -m 1 specifies that the first parent is the one to be reverted. This is necessary because a merge commit has more than one parent.

3. Resolve Conflicts (if any)

If there were changes made in the project after the problematic commit, git revert might introduce conflicts. These need to be resolved manually:

SH
# Stage the resolved files
git add <resolved-files>

# Commit the resolution
git revert --continue
4. Push Changes

Finally, push your reverted changes back to the shared branch. Since you are not rewriting history, other team members can pull these changes seamlessly without issues:

SH
git push origin <branch-name>

#### Closing Summary
and Common Mistakes
Common Mistake: Forgetting to use git revert instead of git reset --hard, which would alter the commit history.
Best Practice: Always prefer git revert for maintaining a clean commit history, especially in shared branches.

#### Example
Workflow

Assume you have the following commit history on your branch:

CODE
A - B - C - D (HEAD) - E - F
              |          |
              +----------+
                   bad-commit-hash
You identify E as the problematic commit.
You run:
SH
git revert -m 1 <bad-commit-hash>
Resolve any conflicts and continue:
SH
git add .
git revert --continue
Push your changes:
SH
git push origin feature-branch

**Code:**

```
SH
Copy
git log --oneline
git revert -m 1 <commit-hash>
git add .
git revert --continue
git push origin <branch-name>
```

