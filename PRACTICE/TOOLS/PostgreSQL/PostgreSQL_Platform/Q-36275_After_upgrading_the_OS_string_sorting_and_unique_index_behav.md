---
type: "practice"
difficulty: "Deep"
---

# After upgrading the OS, string sorting and unique index behavior changed and reindex warnings appeared; explain how a glibc collation version change can corrupt text indexes and how you'd remediate with REINDEX.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Materialized Views vs Incremental Aggregates](<../../../../TOOLS/PostgreSQL/PostgreSQL_Platform/Materialized_Views_vs_Incremental_Aggregates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When upgrading an operating system (OS), particularly one that affects string sorting behavior, such as changes in glibc, it's possible that existing text indexes in PostgreSQL may become corrupted or behave unexpectedly. This is because PostgreSQL relies on the OS's string comparison functions to sort and compare strings correctly. If these functions change due to a glibc upgrade, the internal state of text indexes could be inconsistent with expected ordering.

In this scenario, you would observe reindex warnings indicating that the index structure needs attention. To remediate this issue, you'll need to run REINDEX commands on your affected tables and indexes.

