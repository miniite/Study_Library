---
type: "practice"
difficulty: "Deep"
---

# Two release branches of your model-serving config both need the same latency-optimization fix but the surrounding code diverged. Compare cherry-pick, merge-forward, and a shared base-branch approach for keeping them in sync.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Cherry-picking and Backports](<../../../../TOOLS/Git_and_GitHub/Git_Core/Cherry-picking_and_Backports.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the scenario where two release branches of your model-serving config both need the same latency-optimization fix but the surrounding code diverged, we'll explore three main approaches: cherry-picking, merge-forwarding, and using a shared base-branch approach. Each method has its pros and cons that we'll evaluate from first principles.

TL;DR
Cherry-Pick: Best for isolated changes with minimal context.
Merge-Forward: Useful when the divergent code is significant but still manageable.
Shared Base-Branch: Ideal for large, ongoing development where maintaining a single source of truth is crucial.

#### Triaging the Question

The core question revolves around merging identical fixes into two diverging branches. The interviewer might be probing your understanding of Git workflows and their trade-offs in terms of complexity, risk, and maintainability.

#### Assumptions
Both release branches have substantial differences.
The latency optimization fix is small but critical for all branches.
Merging strategies must balance ease-of-use with minimizing conflicts.

#### First-Principles Solution

Cherry-Pick

Description: Cherry-picking involves applying a specific commit (the fix) from one branch to another without the rest of the changes in that commit.

#### Pros
Simple and fast.
Reduces merge conflicts since only the relevant change is applied.

#### Cons
Requires manual intervention for each fix.
Harder to keep track if multiple similar fixes are needed.

Merge-Forward

Description: Merge-forward involves regularly merging changes from one branch (e.g., main) into both release branches, then cherry-picking the necessary changes back.

#### Pros
Centralizes development on a single mainline, reducing code divergence.
Easier to manage as new features are integrated before branching out.

#### Cons
Increased complexity in maintaining multiple branches.
Potential for more conflicts due to frequent merging.

Shared Base-Branch

Description: Introduce a shared base-branch where both release branches diverge from but periodically merge into, ensuring they stay in sync.

#### Pros
Centralizes development efforts.
Simplifies branch management by reducing divergence.

#### Cons
Requires careful planning and discipline to avoid excessive merge conflicts.
Increases the risk of introducing bugs during merges.
Full Solution
1. Cherry-Pick

Cherry-picking is ideal for minor, isolated changes that don't require context from other commits. For example:

BASH
# Assuming fix commit hash: abcdefg
git checkout release-branch-a
git cherry-pick abcdefg

# Repeat the same process on another branch if needed:
git checkout release-branch-b
git cherry-pick abcdefg
2. Merge-Forward

Merge-forward involves regularly merging changes from a central main or develop branch into both release branches:

BASH
# Merge forward to main
git checkout main
git pull origin main

# Merge into release-branch-a and -b
git checkout release-branch-a
git merge main

git checkout release-branch-b
git merge main

Then, cherry-pick necessary changes back if they are specific:

BASH
git checkout release-branch-a
git cherry-pick <specific-fix-commit>

git checkout release-branch-b
git cherry-pick <specific-fix-commit>
3. Shared Base-Branch

Introduce a shared base-branch where both release branches diverge but periodically merge back to keep in sync:

BASH
# Introduce the shared base-branch
git checkout -b shared-base-branch

# Merge changes from main into shared base branch
git pull origin main

# Split off releases from shared base-branch
git checkout -b release-branch-a
git merge shared-base-branch

git checkout -b release-branch-b
git merge shared-base-branch

# Periodically merge back to keep in sync
git checkout shared-base-branch
git merge main

#### Closing Summary
+ Common Mistakes/Follow-Ups

Summary: Each approach has its strengths and weaknesses. Cherry-picking is simple but requires manual effort, merge-forward centralizes development with less context, and a shared base-branch strategy offers long-term maintainability at the cost of initial complexity.

#### Common Mistakes

Forgetting to cherry-pick fixes: This can lead to out-of-sync branches.
Neglecting to regularly merge forward or into a shared branch: This increases code divergence and potential conflicts.
Over-reliance on manual steps: Can introduce human errors, especially in large teams.

#### Follow-Ups
What are the performance impacts of frequent merges?
How would you handle larger fixes that require context from multiple commits?
In what scenarios might a rebase be more appropriate?

**Code:**

```bash
# Example cherry-pick command
git checkout release-branch-a
git cherry-pick abcdefg

# Example merge-forward process
git checkout main
git pull origin main
git checkout release-branch-a
git merge main
git checkout release-branch-b
git merge main
```

**Example input / output:**

```text
# Cherry-pick example output
$ git checkout release-branch-a
$ git cherry-pick abcdefg

# Merge-forward example output
$ git checkout main
$ git pull origin main
$ git checkout release-branch-a
$ git merge main
```

**Flow:**

1. Main
2. Develop
3. Release-Branch-A
4. Release-Branch-B
5. Shared-Base-Branch

