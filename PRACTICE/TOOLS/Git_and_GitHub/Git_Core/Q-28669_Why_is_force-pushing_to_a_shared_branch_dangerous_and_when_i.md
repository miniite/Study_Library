---
type: "practice"
difficulty: "Deep"
---

# Why is force-pushing to a shared branch dangerous and when is --force-with-lease safer?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reset vs Revert](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reset_vs_Revert.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Force-pushing to a shared branch in Git can be dangerous because it can overwrite or disrupt changes that other team members may have made, potentially leading to lost work. However, using --force-with-lease is safer and more controlled under the right circumstances. Let's break down why force-pushing is risky, when it might be necessary, and how --force-with-lease can help.

#### Triaging the Question

The question asks about the dangers of force-pushing to a shared branch and when --force-with-lease should be used instead. This involves understanding:

What force-pushing does: Overwrites the history on the remote repository.
When it's necessary: In certain scenarios like re-basing, rebasing onto an upstream branch that has moved, or squashing commits for cleaner history.
The risks involved: Overwriting others' work and causing merge conflicts.
--force-with-lease: A safer alternative that checks if the remote branch still exists before overwriting.

#### First Principles Solution
Understanding Force-Pushing

Force-pushing (git push --force) is used to overwrite the history of a branch in the remote repository. This can be dangerous because:

Overwrites History: It replaces all previous commits on the remote branch with new ones, which can erase others' contributions.
Conflict Risks: Other team members might have pushed changes that get lost when you force-push.
Merge Conflicts: Force-pushing can cause merge conflicts if someone else has made changes in a way that doesn't align with your new history.
--force-with-lease

--force-with-lease is a safer variant of the --force option because it includes a mechanism to prevent accidental overwrites. It works by:

Checking Remote State: Before rewriting the history, Git checks if the remote branch has changed.
Lease Mechanism: Ensures that your local copy and the remote repository are in sync before proceeding with force-pushing.
When is --force-with-lease Safer?

--force-with-lease should be used when:

You Need Clean History: You're squashing commits or re-basing onto an upstream branch.
Collaboration Safety: To avoid overwriting others' work, ensuring you only push to the remote if your local copy is in sync with it.

#### Closing Summary
and Common Mistakes

In summary, force-pushing can be dangerous because it overwrites history, potentially causing conflicts or losing other team members’ contributions. --force-with-lease offers a safer alternative by checking the state of the remote branch before rewriting history. It's particularly useful when you need clean history but want to ensure that your local changes align with the remote repository.

#### Common Mistakes
Overusing Force-Pushing: Using it without understanding its implications.
Misunderstanding --force-with-lease: Not realizing that it still has risks if not used correctly (e.g., if you haven't pulled the latest changes).
Ignoring Git Flow Best Practices: Forgetting to use git flow strategies like feature branches and pull requests.

**Code:**

```bash
# Example of force-pushing with --force-with-lease

# Assume local branch 'feature' is ahead of remote 'origin/feature'
git push origin feature --force-with-lease
```

