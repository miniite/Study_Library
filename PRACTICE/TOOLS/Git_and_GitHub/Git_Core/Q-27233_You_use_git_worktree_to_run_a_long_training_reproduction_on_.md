---
type: "practice"
difficulty: "Intermediate"
---

# You use git worktree to run a long training reproduction on one branch while continuing feature work on another. Explain precisely when this beats stashing or a second clone for an ML engineer, and the disk/artifact gotchas.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Git Worktrees](<../../../../TOOLS/Git_and_GitHub/Git_Core/Git_Worktrees.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working as an ML engineer, you often find yourself in situations where you need to run long-running tasks like model training while continuing with other work. Using git worktree is a powerful tool that allows you to achieve this without the overhead of stashing changes or maintaining separate clones of your repository.

To understand when git worktree beats stashing or using multiple clones, let's first look at the key advantages and scenarios where it excels.

**Code:**

```bash
# Example usage of git worktree
git worktree add <path> <branch-name>
```

