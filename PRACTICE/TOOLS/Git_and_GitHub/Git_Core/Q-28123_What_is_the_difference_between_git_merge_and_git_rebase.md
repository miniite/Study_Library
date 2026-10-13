---
type: "practice"
difficulty: "Deep"
---

# What is the difference between git merge and git rebase?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Merge Strategies](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Strategies.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When comparing git merge and git rebase, it's crucial to understand their fundamental differences in how they handle code integration within a Git workflow. Essentially, merge creates a new commit that consolidates changes from two branches into one, whereas rebase integrates the changes from one branch onto another by moving or combining the commits.

To fully grasp these concepts, we need to look at how each operation affects the project's history and potential conflicts.

**Code:**

```
MERMAID
Copy
graph TD;
    A[Master] --> B[Feature];
    C[Feature] --> D[Merge];
    B --> E[Master];
    F[Master] --> G[Feature];
    H[Feature] --> I[Rebase];
    G --> J[Master];
```

**Example input / output:**

```text
Let's take a simple example to illustrate both operations:

**Initial Setup:**
- Master branch has commit A
- Feature branch has commits B and C

**Using `git merge`:**

1. `git checkout feature` - Switch to the feature branch.
2. `git checkout master` - Switch back to the master branch.
3. `git merge feature` - This command creates a new commit that includes both sets of changes, resulting in a merge commit D.

**Using `git rebase`:**

1. `git checkout feature` - Ensure you are on the feature branch.
2. `git rebase master` - This command moves all commits from the feature branch onto the latest version of the master branch, effectively updating the project history to reflect that these changes were always part of the master.

These operations result in different histories:

- **Merge:** A new commit (D) is created on top of E.
- **Rebase:** Commits B and C are moved to the tip of J.
```

**Data:**

```json
{
  "branches": {
    "master": "Initial commit",
    "feature": "Second commit"
  },
  "merge_branches": [
    { "branch": "feature", "action": "merge" },
    { "branch": "master", "new_commit": true, "message": "Merged feature into master" }
  ],
  "rebase_branches": [
    { "branch": "feature", "action": "rebase", "onto": "master", "result": "Updated master with feature history" }
  ]
}
```

