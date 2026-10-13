---
type: "practice"
difficulty: "Deep"
---

# What is the default isolation level in InnoDB and how does REPEATABLE READ work there?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Read Your Writes in MySQL](<../../../../TOOLS/MySQL/MySQL_Platform/Read_Your_Writes_in_MySQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When discussing the default isolation level in MySQL's InnoDB, we need to understand that it operates under a set of predefined transaction isolation levels designed to ensure data consistency while allowing concurrent operations. The default isolation level in InnoDB is REPEATABLE READ (RR), which ensures that once a row has been read within a transaction, that row will always be seen with the same values until the transaction ends.

In this answer, we’ll delve into what REPEATABLE READ means and how it works under the hood. We'll also discuss its implications and common pitfalls.

