---
type: "practice"
difficulty: "Intermediate"
---

# What does the double-dot (main..feature) versus triple-dot (main...feature) syntax mean in git diff and git log?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Reviewing Changes with Diff](<../../../../TOOLS/Git_and_GitHub/Git_Core/Reviewing_Changes_with_Diff.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing git diff and git log, the double-dot (..) and triple-dot (...) syntaxes are used to specify different ranges of commits, allowing developers to compare branches or paths effectively. In this context:

Double-dot (main..feature) is often used in git diff and git log commands to show changes between two named branches.
Triple-dot (main...feature) is typically used for complex comparisons that involve more than two branches.

Let's dive into how these work step by step, from first principles.

**Code:**

```bash
# Example usage of git diff with double-dot and triple-dot syntax
git diff main..feature  # Shows changes between 'main' and 'feature'
git log main...feature  # Shows commit history that is in feature but not in main
```

**Flow:**

1. No
2. Yes
3. main
4. Initial State
5. Is feature branch merged intomain?
6. Show all changes betweenmain and feature
7. Use git diff main...feature toshow unique history in featurenot in main

