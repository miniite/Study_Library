---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/Temporal_and_Graph_Tables"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Temporal and Graph Tables

**L4 Advanced** · [↑ T-SQL Programming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** System-versioned temporal tables make SQL Server keep every previous version of each row automatically in a history table, so you can query the data "as of" any point in time. Graph tables (node and edge tables with `MATCH`) model many-to-many relationships and paths inside the relational engine.

### 1. The big picture
Two recurring requirements are awkward in plain tables:
1. **History**: "What did this customer's credit limit look like last March?" "Who changed this price and when?" Hand-built audit tables and triggers are error-prone.
2. **Networks**: "Who are the friends of my friends?" "Which parts does this assembly contain, at any depth?" Recursive CTEs over self-referencing tables work but get complex.

SQL Server 2016 added **system-versioned temporal tables** (from the SQL:2011 standard) for history, and SQL Server 2017 added **SQL Graph** for networks. Both are regular tables underneath, so they work with indexes, backups, security and T-SQL.

### 2. Core concepts and vocabulary
**Temporal tables**
- **System-versioned temporal table**: a *current* table plus a linked *history* table. SQL Server moves old row versions into history on every UPDATE and DELETE.
- **Period columns**: two `datetime2` columns marked `GENERATED ALWAYS AS ROW START` / `ROW END`, plus `PERIOD FOR SYSTEM_TIME (start, end)`. The engine sets them; you can't write them.
- **Validity interval**: the half-open interval `[ValidFrom, ValidTo)` during which a row version was current. Current rows have `ValidTo = 9999-12-31 23:59:59.9999999`.
- **UTC**: period values are stored in UTC, and they record the *transaction begin time*.
- **`FOR SYSTEM_TIME`**: query clause to read history: `AS OF`, `FROM … TO`, `BETWEEN … AND`, `CONTAINED IN`, `ALL`.
- **HIDDEN**: option that hides period columns from `SELECT *`.
- **History retention**: `HISTORY_RETENTION_PERIOD` automatically cleans up old history (Azure SQL, SQL Server 2017+).
- **System time vs. application time**: SQL Server supports *system* time (when the database knew it). *Application/business* time ("valid in the real world from…") you model yourself.

**Graph tables**
- **Node table** (`AS NODE`): entities (Person, City). Gets a hidden `$node_id`.
- **Edge table** (`AS EDGE`): relationships (FriendOf, LivesIn). Gets `$edge_id`, `$from_id`, `$to_id`; may have its own attributes (e.g. `since`).
- **`MATCH`**: predicate in `WHERE` describing a pattern with ASCII-art arrows: `MATCH(p1-(f)->p2)`.
- **`SHORTEST_PATH`** (SQL 2019): finds shortest paths of arbitrary length, with `STRING_AGG` / `LAST_VALUE` over the path.
- **Edge constraint** (SQL 2019): restricts which node types an edge can connect.

### 3. How it works, step by step
**Temporal: what happens on each DML statement**

| Statement | Current table | History table |
|---|---|---|
| INSERT | New row, `ValidFrom` = transaction start, `ValidTo` = max | Nothing |
| UPDATE | Row updated, `ValidFrom` = transaction start | Old version inserted, `ValidTo` = transaction start |
| DELETE | Row removed | Old version inserted, `ValidTo` = transaction start |

**How `AS OF @t` is answered:** the engine reads the current table where `ValidFrom <= @t` (and `ValidTo > @t`, always true for current rows), unions it with history rows where `ValidFrom <= @t AND ValidTo > @t`. The plan shows both tables and a *Concatenation*.

**Sub-clauses:**

| Clause | Returns rows whose interval… |
|---|---|
| `AS OF t` | contains t (`ValidFrom <= t < ValidTo`) |
| `FROM a TO b` | overlaps `[a, b)` (`ValidFrom < b AND ValidTo > a`) |
| `BETWEEN a AND b` | overlaps, including rows that start exactly at b |
| `CONTAINED IN (a, b)` | lies fully inside `[a, b]` |
| `ALL` | everything, current and history |

**Graph: what happens**
1. Node and edge tables are normal tables with extra hidden columns (JSON-like IDs internally mapped to object ID + integer IDs).
2. `MATCH` patterns are translated into joins between node and edge tables on `$from_id`/`$to_id`.
3. `SHORTEST_PATH` with `+` (one or more hops) or `{1,n}` runs an iterative breadth-first search.

### 4. Worked example
**Temporal table:**
```sql
CREATE TABLE dbo.Product
(
    ProductID  int           NOT NULL PRIMARY KEY CLUSTERED,
    Name       nvarchar(100) NOT NULL,
    ListPrice  decimal(10,2) NOT NULL,
    ValidFrom  datetime2(7) GENERATED ALWAYS AS ROW START HIDDEN NOT NULL,
    ValidTo    datetime2(7) GENERATED ALWAYS AS ROW END   HIDDEN NOT NULL,
    PERIOD FOR SYSTEM_TIME (ValidFrom, ValidTo)
)
WITH (SYSTEM_VERSIONING = ON (HISTORY_TABLE = dbo.ProductHistory,
                              HISTORY_RETENTION_PERIOD = 2 YEARS));

INSERT dbo.Product (ProductID, Name, ListPrice) VALUES (1, N'Widget', 10.00);
-- later…
UPDATE dbo.Product SET ListPrice = 12.00 WHERE ProductID = 1;
-- later…
UPDATE dbo.Product SET ListPrice = 11.50 WHERE ProductID = 1;

-- Full price history of product 1
SELECT ProductID, ListPrice, ValidFrom, ValidTo
FROM dbo.Product FOR SYSTEM_TIME ALL
WHERE ProductID = 1
ORDER BY ValidFrom;

-- What the table looked like at a moment (period values are UTC)
DECLARE @t datetime2 = '2026-10-01T09:00:00';
SELECT ProductID, Name, ListPrice
FROM dbo.Product FOR SYSTEM_TIME AS OF @t;

-- Recover a row deleted by mistake
INSERT dbo.Product (ProductID, Name, ListPrice)
SELECT ProductID, Name, ListPrice
FROM dbo.Product FOR SYSTEM_TIME AS OF @t
WHERE ProductID = 1;
```

Converting an existing table: add the period columns with defaults (`DEFAULT SYSUTCDATETIME()` and `DEFAULT CONVERT(datetime2, '9999-12-31 23:59:59.9999999')`), add `PERIOD FOR SYSTEM_TIME`, then `ALTER TABLE … SET (SYSTEM_VERSIONING = ON (HISTORY_TABLE = …))`.

**Graph tables:**
```sql
CREATE TABLE dbo.Person (PersonID int PRIMARY KEY, Name nvarchar(50)) AS NODE;
CREATE TABLE dbo.FriendOf (Since date) AS EDGE;

INSERT dbo.Person VALUES (1,N'Ana'),(2,N'Ben'),(3,N'Cy'),(4,N'Di');

-- Edges: Ana->Ben, Ben->Cy, Cy->Di (look up both $node_id values per pair)
INSERT dbo.FriendOf ($from_id, $to_id, Since)
SELECT f.$node_id, t.$node_id, '20200101'
FROM (VALUES (1,2), (2,3), (3,4)) AS pairs(fromid, toid)
JOIN dbo.Person AS f ON f.PersonID = pairs.fromid
JOIN dbo.Person AS t ON t.PersonID = pairs.toid;

-- Friends of friends of Ana
SELECT p3.Name
FROM dbo.Person p1, dbo.FriendOf f1, dbo.Person p2, dbo.FriendOf f2, dbo.Person p3
WHERE MATCH(p1-(f1)->p2-(f2)->p3)
  AND p1.Name = N'Ana';

-- Shortest path from Ana to anyone, at any depth (SQL 2019+)
SELECT p1.Name,
       STRING_AGG(p2.Name, '->') WITHIN GROUP (GRAPH PATH) AS path,
       LAST_VALUE(p2.Name)       WITHIN GROUP (GRAPH PATH) AS reached
FROM dbo.Person AS p1,
     dbo.FriendOf FOR PATH AS f,
     dbo.Person   FOR PATH AS p2
WHERE MATCH(SHORTEST_PATH(p1(-(f)->p2)+))
  AND p1.Name = N'Ana';
```
The friends-of-friends query returns Cy (Ana → Ben → Cy). The shortest-path query returns one row per reachable person with its path, e.g. `Ben->Cy->Di` for Di.

### 5. Under the hood
- **Same transaction.** The history insert happens inside the DML transaction, so history is always consistent; it also means updates do roughly double the writes.
- **Times are transaction begin times.** All rows changed in one transaction get the same timestamp, even if the transaction ran for minutes. Updating the same row twice in one transaction produces a zero-length history version (`ValidFrom = ValidTo`), which `AS OF` never returns.
- **History table design.** By default the history table gets a clustered index on (`ValidTo`, `ValidFrom`) and page compression. For large history, a **clustered columnstore** is a good choice. History is read-only to users while versioning is on.
- **Schema changes** (adding columns) propagate to history automatically. Some operations (e.g. `TRUNCATE`, certain ALTERs) require turning versioning off first: `SET (SYSTEM_VERSIONING = OFF)`, change, turn back on with `DATA_CONSISTENCY_CHECK`.
- **Retention cleanup** runs as a background task deleting aged history rows in batches; with columnstore history it drops whole row groups.
- **Graph internals.** `$node_id`/`$edge_id` are computed JSON strings over internal `graph_id` bigint columns. Create indexes on edge tables on `($from_id, $to_id)` and `($to_id, $from_id)`, or every `MATCH` scans the edge table. The optimizer turns MATCH into ordinary joins, so normal plan analysis applies.

### 6. Performance and design implications
- Temporal adds write cost (one history row per update/delete) and storage. Tables with very frequent updates of wide rows produce large history; use retention and columnstore.
- Queries on current data are unaffected; `AS OF` queries touch both tables, so index the history table for your access pattern (e.g. `(ProductID, ValidTo, ValidFrom)`).
- Temporal history is not an audit of *who* changed data. Add a `ModifiedBy` column set by the application or `SESSION_CONTEXT`, or use SQL Audit.
- Graph is good for moderate graphs and simple pattern queries inside an OLTP database. For very large graph analytics (PageRank, community detection), a dedicated graph engine is more suitable.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PERIOD FOR SYSTEM_TIME`, `GENERATED ALWAYS AS ROW START/END` | Defines period columns | Creating temporal tables |
| `SYSTEM_VERSIONING = ON (HISTORY_TABLE = …)` | Turns on versioning | Create / convert |
| `HISTORY_RETENTION_PERIOD = n MONTHS` | Auto-cleanup | Limit history growth |
| `FOR SYSTEM_TIME AS OF / FROM / BETWEEN / CONTAINED IN / ALL` | Time travel queries | Point-in-time reports, recovery |
| `SET (SYSTEM_VERSIONING = OFF)` | Detach history | Schema maintenance |
| `sys.tables.temporal_type_desc` | Lists temporal tables | Inventory |
| `AS NODE` / `AS EDGE` | Graph tables | Network modeling |
| `MATCH`, `SHORTEST_PATH` | Graph pattern queries | Friends-of-friends, routes |
| `CONSTRAINT … CONNECTION (A TO B)` | Edge constraint | Data quality in graphs |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Temporal for point-in-time reporting, accidental-change recovery, slowly changing dimension sources | You need business-effective dates → model application time yourself |
| Temporal instead of audit triggers | You need "who" → add columns or use SQL Audit |
| Columnstore history | Small history: rowstore is fine |
| Graph for many-to-many networks with path queries | Simple hierarchies → `hierarchyid` or a parent column with recursive CTE |
| Recursive CTE | Arbitrary-length shortest paths → `SHORTEST_PATH` |

### 9. Common mistakes
- Passing local time to `AS OF` → off by the UTC offset → convert with `AT TIME ZONE`.
- Expecting `ValidFrom` to be the statement time → it's transaction start → keep transactions short if time precision matters.
- Letting history grow forever → storage and backup growth → retention period.
- Assuming temporal tells you who made the change → it doesn't → add a user column.
- No indexes on edge `$from_id`/`$to_id` → slow MATCH queries.
- Using `TRUNCATE` on a temporal table → not allowed → switch versioning off or delete.

### 10. 🎯 Interview kit
**30-second answer:** A system-versioned temporal table has period columns maintained by SQL Server and a linked history table. Every update or delete copies the old version to history in the same transaction, with UTC transaction-start timestamps. FOR SYSTEM_TIME AS OF, FROM/TO, BETWEEN, CONTAINED IN and ALL let you query any point or range in time. Graph tables add node and edge tables with MATCH and SHORTEST_PATH for network queries.

**Structure a longer answer:** the requirement (history or networks) → mechanics (what each DML does) → query clauses → storage/retention design → limitations (UTC, transaction time, no "who") → a real use (recovering deleted rows, SCD feed).

**Likely follow-ups:**
- *How does AS OF work internally?* → union of current and history filtered on the validity interval.
- *How to control history size?* → retention period, columnstore, partitioning.
- *Temporal vs. CDC?* → temporal = queryable versions; CDC = change stream from the log for ETL.
- *Graph vs. recursive CTE?* → MATCH patterns and SHORTEST_PATH vs. manual recursion.

**Signals interviewers look for:** you know period semantics (half-open, UTC, transaction time); you design for history growth; you compare with CDC/audit correctly.

### 11. Check yourself
- [ ] Create a temporal table, make three updates, and list the full history.
- [ ] Recover a deleted row with `AS OF`.
- [ ] Explain the difference between `FROM … TO` and `CONTAINED IN`.
- [ ] Convert an existing table to temporal.
- [ ] Build a tiny graph and query friends-of-friends with MATCH.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Temporal Tables and System-Versioned Data
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Understanding Temporal Tables
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › System-Versioned Data Mechanics
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Real-World Example
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Querying Temporal Data
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Best Practices for Temporal Tables
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Key Takeaways
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Use Cases for Temporal Tables
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Temporal tables
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 15: Case Studies and the Future of T-SQL › Integrating with ETL and Reporting

