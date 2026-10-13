---
type: "practice"
difficulty: "Deep"
---

# You need to back-port a data-correctness fix to a stored procedure across five older release tags where the surrounding package diverged. Walk through cherry-pick -x for provenance, rebase --onto for a range, and format-patch/am for offline transfer, and how you resolve per-tag conflicts.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Cherry-picking and Backports](<../../../../TOOLS/Git_and_GitHub/Git_Core/Cherry-picking_and_Backports.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To back-port a data-correctness fix to a stored procedure across five older release tags with divergent surrounding packages, we need to use Git commands that allow us to selectively apply changes while handling potential conflicts. The key steps involve using cherry-pick -x for provenance, rebase --onto for range-based operations, and format-patch/am for offline transfer.

Firstly, it’s important to understand the context: we are working with multiple release tags that have diverged from a shared branch. The goal is to bring a specific fix (e.g., in a stored procedure) to these older releases without introducing conflicts or unnecessary complexity.

**Code:**

```bash
# Example commands for cherry-pick, rebase --onto, and format-patch/am
git cherry-pick -x <commit-hash>  # Applies the commit while tracking its origin
git checkout <target-branch>      # Switch to target branch where backport is needed
git rebase --onto <new-base> <old-base> <commit-range>  # Rebases a range of commits onto a new base
git format-patch -M -B <patch-id> # Creates patches for sending or applying offline
```

**Flow:**

1. Initial Commit
2. (cherry-pick -x<commit-hash>)
3. Apply to Target Branch
4. (git checkout<target-branch>)
5. Rebase Range of Commits
6. (git rebase --onto <new-base><old-base> <commit-range>)
7. Handle Conflicts if Any
8. (format-patch/am for offlinetransfer)

