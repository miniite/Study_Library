---
type: "practice"
difficulty: "Deep"
---

# Describe a case where you chose a MERGE statement over separate INSERT/UPDATE logic in PL/SQL and how you handled the WHEN NOT MATCHED and DELETE clauses.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ PL-SQL in Production](<../../../../TOOLS/Oracle/PL-SQL/PL-SQL_in_Production.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, especially when dealing with large datasets or complex data synchronization tasks, choosing the right SQL statement can significantly impact performance and maintainability. One situation where I opted for a MERGE statement over separate INSERT/UPDATE logic was during an application upgrade that required migrating user settings from an old system to a new one.

The core idea behind using MERGE in this context is its ability to consolidate the operations of INSERT, UPDATE, and even DELETE into a single, more efficient statement. This not only simplifies the code but also reduces the risk of data inconsistencies that can occur with multiple statements.

In this scenario, I was tasked with handling three main cases:

When a record exists in both systems (new system and old system).
When a record does not exist in the new system but does in the old one.
When a record needs to be deleted from the new system if it no longer exists in the old system.

Here’s how I handled each of these cases using MERGE:

When a Record Exists in Both Systems:

The primary key of the user settings is used as a unique identifier.
If the record matches on this key, an UPDATE operation is performed to synchronize any changes from the old system to the new one.

When a Record Does Not Exist in the New System but Exists in the Old One:

The WHEN NOT MATCHED BY TARGET clause ensures that records present only in the old system are inserted into the new system.

When a Record Needs to Be Deleted from the New System:

The WHEN NOT MATCHED BY SOURCE clause handles the case where records exist in the new system but not in the old one, allowing for their removal.

