---
type: "practice"
difficulty: "Intermediate"
---

# How would you undo the last commit that added a wrong DROP TABLE statement, but keep the file so you can fix it, before pushing?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Undoing Commits](<../../../../TOOLS/Git_and_GitHub/Git_Core/Undoing_Commits.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To undo the last commit that contained an incorrect DROP TABLE statement in Git without losing the changes, we need to follow a few steps carefully. The core idea here is to revert the commit and then fix the issue before committing it again. Let’s break down the process step by step.

Firstly, you should understand what happened: someone mistakenly added a DROP TABLE command, which would delete an important table if executed. You need to undo this change without losing the ability to modify the file so that you can rectify the mistake before pushing it to the remote repository.

