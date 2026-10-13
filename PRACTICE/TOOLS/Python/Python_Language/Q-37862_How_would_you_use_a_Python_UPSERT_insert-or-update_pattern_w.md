---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a Python UPSERT (insert-or-update) pattern with an ON CONFLICT or MERGE statement to keep a target table in sync with a source?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Streaming Query Results](<../../../../TOOLS/Python/Python_Language/Streaming_Query_Results.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure that a target database table stays in sync with a source using Python, we can leverage the UPSERT (insert-or-update) pattern, which is effectively implemented by the ON CONFLICT statement in SQL or through MERGE statements. This approach ensures that if a record exists in both the source and the target tables, it gets updated; otherwise, it gets inserted into the target.

In this explanation, I will start with a brief overview of how to use the UPSERT pattern, then delve into the steps involved from first principles using Python and PostgreSQL (as an example). Finally, I'll summarize common pitfalls to avoid.

