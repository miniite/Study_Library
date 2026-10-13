---
type: "practice"
difficulty: "Deep"
---

# During an incident you must identify every commit that touched a specific critical function across renames and refactors in the last year. Which combination of log --follow, -L, and pickaxe would you use and why?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Searching History](<../../../../TOOLS/Git_and_GitHub/Git_Core/Searching_History.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this incident response scenario, we need to find every commit that touched a specific critical function across renames and refactors in the last year. This involves leveraging Git's powerful log command along with its associated flags and pickaxe commands. Here’s how you can approach it:

Firstly, let's understand what each component does:

git log --follow: Tracks history of a file even when the name changes (renames).
git log -L: Lines-based searching for specific text patterns.
git log --pickaxe-commit: Shows only commits where a particular line is added or removed.

Given these tools, we can construct an effective command to find the required commits. The combination of --follow, -L, and --pickaxe-commit will help us achieve our goal.

