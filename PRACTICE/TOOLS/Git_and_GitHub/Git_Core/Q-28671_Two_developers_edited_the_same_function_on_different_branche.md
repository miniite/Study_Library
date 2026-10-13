---
type: "practice"
difficulty: "Intermediate"
---

# Two developers edited the same function on different branches. How does the merge behave and how would you resolve it?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Merge Conflicts](<../../../../TOOLS/Git_and_GitHub/Git_Core/Merge_Conflicts.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When two developers make changes to the same function in a Git repository, they are working on separate branches. During the merge process, Git tries to combine these changes into a single commit. However, if both developers modify the same lines of code (a conflict), Git cannot automatically resolve it and requires manual intervention.

The core idea is that the merge behavior depends on how Git handles these conflicts, which can be resolved through either merging or rebasing strategies. The key steps involve identifying conflicts, understanding their nature, and resolving them appropriately to maintain the integrity and functionality of the code.

**Comparison:**

```
Conflict Type	Resolution Strategy
Text-based	Interactive
Binary files	Automated
```

**Code:**

```bash
# Example Git commands for the scenario
git fetch origin  # Fetch latest changes from remote repository
git checkout main  # Switch to the main branch or a relevant one
git merge feature_branch  # Merge the feature branch into the main branch
```

**Example input / output:**

```
Initial state

Feature branch:

PYTHON
Copy
def calculate_sum(a, b):
    return a + b  # Developer A adds an additional parameter

# Main branch:
def calculate_sum(a, b):  # Developer B modifies the function signature
    return a - b
Merge command and conflict output

$ git checkout main $ git merge feature/bug-fix Auto-merging app.js CONFLICT (content): Merge conflict in app.js
```

**Data:**

```text
- Feature branch: `feature/bug-fix` (Developer A)
- Main branch: `main` (Developed B)
```

**Flow:**

1. Yes
2. No
3. Start
4. Merge branches
5. Are there conflicts?
6. Resolve conflicts manually
7. Final commit and push toremote
8. Use Git GUI or command linetools to resolve
9. Stage changes
10. Create final merge commit
11. Pull request review if needed
12. Push to remote repository

