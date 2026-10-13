---
type: "practice"
difficulty: "Expert"
---

# How do implicit cursor attributes like SQL%BULK_ROWCOUNT behave after a FORALL, and how do you use them for per-element counts?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Bulk Collect and FORALL](<../../../../01_Databases_and_SQL/Database_Objects/Cursors/Bulk_Collect_and_FORALL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PL/SQL, cursors are fundamental tools used to handle database operations, especially when dealing with large datasets. An FORALL statement is a powerful feature that allows processing multiple rows in bulk without executing a separate SQL statement for each row. However, after using an FORALL, some cursor attributes behave differently than expected due to how PL/SQL handles bulk operations internally. This answer will explain the behavior of implicit cursor attributes like SQL%BULK_ROWCOUNT and provide examples on how to use them effectively.

