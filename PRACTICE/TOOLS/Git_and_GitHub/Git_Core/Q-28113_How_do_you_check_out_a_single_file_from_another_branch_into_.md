---
type: "practice"
difficulty: "Basic"
---

# How do you check out a single file from another branch into your current working tree without switching branches?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fetch vs Pull](<../../../../TOOLS/Git_and_GitHub/Git_Core/Fetch_vs_Pull.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To check out a single file from another branch into your current working tree without switching branches, you can use Git commands to fetch and apply changes from that specific file. This is useful when you want to merge only part of one branch's work into your current branch. Here’s how you can achieve this:

First, understand the basic idea: Git allows granular operations on files across branches without fully switching branches. You'll need to use git checkout in conjunction with --patch or --theirs options.

Now let’s break down the process step-by-step:

**Code:**

```bash
# Step 1: Make sure you are on your current branch
git checkout <current-branch>

# Step 2: Use git show to inspect the file from the other branch
git show <other-branch>:<path-to-file> > <path-to-file>

# Alternatively, using --theirs with git checkout
git checkout <other-branch> -- <path-to-file>
```

**Example input / output:**

```plaintext
Given:
  - Current branch: develop
  - Other branch: feature/new-feature

To check out "src/utils.js" from the `feature/new-feature` branch into the current working tree on `develop`.

Steps to achieve this:

1. Ensure you are on your current branch (e.g., develop).
2. Run:

git checkout feature/new-feature -- src/utils.js

CODE
Copy

After running these commands, "src/utils.js" from the `feature/new-feature` branch will be checked out into the working tree of `develop` without switching branches.
```

