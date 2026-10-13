---
type: "practice"
difficulty: "Expert"
---

# What is table bloat / dead tuples in an MVCC database and why does VACUUM matter?

**Difficulty:** Expert · **Levels:** Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ MVCC Bloat and Vacuum](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/MVCC_Bloat_and_Vacuum.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Table bloat and dead tuples are critical concepts in databases that use Multi-Version Concurrency Control (MVCC), such as PostgreSQL. Essentially, table bloat refers to excessive disk space usage due to large numbers of dead rows or pages, while dead tuples are rows in a database that have been marked for deletion but haven't yet been removed by the garbage collector. VACUUM is an essential maintenance operation that cleans up these dead tuples, reclaiming space and optimizing query performance.

**Code:**

```text
-- Example of a simple VACUUM command
VACUUM my_table;
```

**Example input / output:**

```text
Before VACUUM:
- Table size: 10 GB, but only 2 GB is effectively used (8 GB is dead tuples).
- Query performance degradation due to excessive number of physical reads.

After VACUUM:
- Table size: ~2 GB.
- Improved query performance due to reduced number of physical reads and better use of available space.
```

**Flow:**

1. Transaction Writes
2. Marked for Deletion
3. Space Occupied
4. VACUUM Runs
5. Dead Tuples Removed
6. Space Reclaimed
7. Improved Query Performance

