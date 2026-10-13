---
type: "practice"
difficulty: "Deep"
---

# A LINQ query that worked in memory throws or silently pulls all rows once moved to IQueryable against EF Core — how do you catch client-side evaluation before it reaches production?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Entity Framework Core Performance](<../../../../TOOLS/DotNet/DotNet_Platform/Entity_Framework_Core_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of a LINQ query working correctly in-memory but pulling all rows when converted to IQueryable and used with Entity Framework Core (EF Core), we need to understand how LINQ to Objects (in-memory queries) versus IQueryable (lazy-loaded database queries) behave. The key is to ensure client-side evaluation of the LINQ query before it reaches production, which can be achieved by explicitly forcing a client-side execution using methods like ToList(), ToArray(), or similar.

Summary: The problem occurs because LINQ to Objects evaluates queries immediately in-memory, while IQueryable waits for execution at the database level. To prevent pulling all rows from the database, we need to force evaluation of the query on the client side before it hits production.

