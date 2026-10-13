---
type: "practice"
difficulty: "Deep"
---

# Can you COMMIT or ROLLBACK inside a trigger? Why or why not?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Trigger Types](<../../../../TOOLS/Oracle/PL-SQL/Trigger_Types.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing PL/SQL and triggers, one often wonders about the behavior of COMMIT and ROLLBACK. The answer is that COMMIT and ROLLBACK operations are generally not allowed inside a trigger in PL/SQL because they would violate the fundamental purpose and design constraints of database triggers. In this response, we'll explore why this restriction exists and what happens when you attempt to include such commands.

