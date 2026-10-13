---
type: "practice"
difficulty: "Deep"
---

# What are the dangers of amending or rebasing a commit that has already been pushed to a shared branch?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reset vs Revert](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reset_vs_Revert.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git and GitHub, one must be cautious about amending or rebasing commits that have already been pushed to a shared branch. This practice can introduce several risks, including losing commit history, causing merge conflicts, and disrupting team collaboration. Let’s explore these dangers in detail.

To understand the issue fully, we need to first clarify what amending and rebasing entail:

Amending (using git commit --amend or git rebase -i HEAD~n) modifies a recent commit.
Rebasing (using git rebase <branch> or interactive rebase) changes the base of a branch, effectively rewriting history.

#### Triaging the Question

The question is asking about the potential pitfalls of amending or rebasing a commit that has already been pushed to a shared branch. The interviewer likely wants you to discuss how these operations can affect other team members and the overall project history.

#### Assumptions
The shared branch is actively used by multiple developers.
The commit in question has already been pushed and possibly included in pull requests or deployments.

#### First Principles Solution

Losing Commit History

When you amend a commit, Git creates a new commit with the same message but updated content. This new commit effectively replaces the old one. If this amended commit is part of a shared branch that has already been pushed and merged, other team members may have based their work on the original commit.
For example, consider a scenario where commit A was originally committed and pushed to main, then a bug was found in it. If you amend A (creating A') without informing others, any changes they made based on A will now conflict with the new history.

Merge Conflicts

Rebasing a branch that has been pushed can lead to merge conflicts when attempting to rebase onto a base that has diverged significantly from your original commit. This is because Git will try to apply all changes from the current HEAD as if they were made on top of main, which could result in conflicts.
For instance, suppose you rebased feature-branch onto main. If multiple developers have pushed new commits to main since your feature branch was created, attempting to rebase and merge can lead to complex merges or even failures.

Disrupting Team Collaboration

Amending a commit that has been part of pull requests or deployments can cause significant disruptions in the workflow.
If a commit with critical changes is pushed to main and then amended, it may break the integrity of any dependent work. For example, if multiple tests or automated builds are based on the original state of the codebase, amending could render these tests obsolete or incorrect.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, while amending and rebasing can be powerful tools for cleaning up your commit history, they should be used judiciously, especially in shared branches. To mitigate risks:

Always communicate with the team before making such changes.
Use git push --force or git push --force-with-lease only when absolutely necessary to avoid conflicts.

#### Cons
ider using a new branch for critical amendments and merges.

#### Common mistakes
include:

Failing to notify team members before amending/rebasing shared commits.
Using force-pushing without understanding the impact on dependent work.
Overusing rebasing, which can complicate the history unnecessarily.

#### Follow-up
questions could involve how to handle such situations effectively or discussing best practices for managing commit history in shared branches. For example:

How would you handle a situation where someone has already based their work on an amended commit?
What are some strategies to avoid these pitfalls altogether?

**Code:**

```
MARKDOWN
Copy
# Example of amending a commit
git commit --amend

# Example of rebasing a branch
git rebase -i <commit-hash>
```

**Flow:**

1. Yes
2. No
3. Start
4. Amend/Rebase Commit?
5. Notify Team
6. Perform Operation
7. Check History
8. Commit New Changes

