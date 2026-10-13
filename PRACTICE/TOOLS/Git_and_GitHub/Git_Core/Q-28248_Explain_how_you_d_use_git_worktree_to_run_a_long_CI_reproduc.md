---
type: "practice"
difficulty: "Intermediate"
---

# Explain how you'd use git worktree to run a long CI reproduction on one branch while continuing feature work on another, and when this beats stashing or a second clone.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Git Worktrees](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Worktrees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with Git, managing multiple branches simultaneously can be challenging, especially when you need to run lengthy CI builds (Continuous Integration) on one branch while continuing development work on another. One powerful tool in this scenario is git worktree, which allows you to have separate working trees for a single repository. This method avoids the overhead of stashing or creating an additional clone of the repository, making your workflow more efficient.

To illustrate how we can use git worktree effectively, let's break down the steps and understand its benefits compared to using stashes or multiple clones.

**Code:**

```bash
# First, ensure you have git version 2.14+ installed which supports `worktree`
$ git --version

# Create a new branch for feature work if not already done
$ git checkout -b feature-branch origin/main

# List current working trees
$ git worktree list

# Create a new worktree for the long CI reproduction on another branch, e.g., main
$ git worktree add /path/to/new/worktree/ main

# You can now run your lengthy CI build in this directory without affecting your feature branch
cd /path/to/new/worktree/
git checkout main  # Ensures you are working with the correct branch
make ci-build     # Your CI build command here
```

**Example input / output:**

```

Assuming you're running a lengthy CI build on main:

PLAINTEXT
Copy
# In your new worktree directory
/path/to/new/worktree/
$ make ci-build  # This could take several minutes or hours

In parallel, back in your original working tree where you are doing feature work:

BASH
Copy
# Go back to the feature-branch and continue development
cd /path/to/original/working/tree/
git checkout feature-branch
# Make changes and commit as usual
git add .
git commit -m "Implemented feature XYZ"
```

**Flow:**

1. Feature Branch Work
2. Checkout Main Branch in NewWorktree
3. Run CI Build in/path/to/new/worktree/
4. Continue Development onFeature Branch
5. Make Changes, Commit
6. Switch Back to Main Branch ifNeeded

