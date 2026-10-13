---
id: "TOOLS/SQL_Server/02_Indexing/Clustered_Indexes"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Clustered Indexes

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A clustered index *is* the table — its data pages arranged as a B-tree in clustering-key order — so the choice of clustering key shapes storage, insert behavior, fragmentation and the size of every other index on the table.

### 1. The big picture
A table is stored either as a heap (no order) or as a clustered index. In a clustered index the leaf level contains the complete rows, sorted by the clustering key, and the upper levels let SQL Server navigate to any key in a few page reads. Because rows can only be stored in one order, a table has at most one clustered index.

The clustered index matters far beyond its own queries:
- It is the **row locator** for every nonclustered index: each NC index row carries the clustering key, and each key lookup navigates the clustered index with it.
- Its key pattern decides **where inserts land** — at the end (sequential) or all over the table (random, causing splits).
- It decides **how ranges and sorts** on that key are served — as one contiguous read with no sort operator.

So clustering-key design is a table-design decision, not just a tuning step. The parent lesson gave the map of index types; the sibling *Nonclustered Indexes* lesson builds on the row-locator idea introduced here; B-tree mechanics and page splits were covered in the storage module.

### 2. Core concepts and vocabulary
- **Clustering key** — the column(s) defining the clustered index order. Max 16 columns, 900 bytes.
- **Leaf level** — the data pages themselves; each leaf row is a full table row.
- **Root / intermediate levels** — index pages holding (key, child page) pairs used for navigation.
- **Logical order** — rows are ordered by key through the page chain and slot arrays; pages are not guaranteed to be physically contiguous on disk (that's what fragmentation measures).
- **Uniquifier** — a hidden 4-byte integer SQL Server adds to duplicate clustering key values in a non-unique clustered index so every row is uniquely addressable.
- **Primary key** — a constraint (unique, NOT NULL). By default it creates a clustered index *if none exists yet*; otherwise it creates a nonclustered one.
- **Unique index / unique constraint** — enforces uniqueness; unlike a PK, allows NULL — but only one NULL per unique index, since NULLs are treated as equal for this purpose.
- **Ever-increasing key** — each new value sorts after all existing ones (IDENTITY, SEQUENCE, insertion datetime).
- **Static key** — a value that never (or very rarely) changes after insert.
- **Surrogate key** — a key with no business meaning (IDENTITY, GUID); a **natural key** has business meaning (ISBN, state code).

### 3. How it works, step by step
**Reading via the clustered index**
1. *Seek:* start at the root (location kept in metadata), binary-search its slot array for the key range, follow the child pointer, repeat until the leaf. A table of millions of rows is typically 3–4 levels deep → 3–4 logical reads per singleton lookup.
2. *Range scan:* after seeking to the first key, follow the leaf-level next-page pointers until the range ends. Rows come out in key order, so `ORDER BY` on the key needs no sort.
3. *Full scan:* read all leaf pages (in key order, or in allocation order when an unordered scan is allowed).

**Writing to the clustered index**
1. Navigate to the leaf page where the new key belongs.
2. If there is room, insert the row and its slot entry.
3. If not, **split** the page: allocate a new page, move about half the rows (or, for an insert at the very end of the index, just start a new page), fix the prev/next pointers and the parent entry. Splits are logged and can cascade upward.
4. Update every nonclustered index affected.

**Key properties of a well-defined clustering key** (the classic four):

| Property | Why it matters |
|---|---|
| **Narrow** | Copied into every NC index row and every non-leaf row; 4 bytes vs 16 bytes multiplies across billions of rows |
| **Unique** | Avoids the 4-byte uniquifier and makes each row's locator unambiguous |
| **Static** | Changing a clustering key = delete + insert (row moves) and updates every NC index |
| **Ever-increasing** | Inserts go to the end of the B-tree → no mid-index splits, little fragmentation, hot pages stay in cache |

**Common patterns for choosing a key**
- **Identity column** — add an `INT`/`BIGINT IDENTITY` (or SEQUENCE) and cluster on it. Narrow, unique, static, ever-increasing. The default choice for most OLTP tables.
- **Natural / short code key** — e.g. `CHAR(2)` state code or `CHAR(1)` status. Narrower than an INT and readable; not ever-increasing, but for small, rarely inserted lookup tables that doesn't matter.
- **Foreign key pattern** — cluster a child table on (ParentID, ChildID), e.g. order lines on (OrderID, LineID). All rows of one parent are stored together, so "get all lines for an order" is one range read. Wider (8 bytes for two INTs) and only roughly ever-increasing (late detail rows for old headers land mid-index).
- **Date/time pattern** — cluster on an insert or event date when most queries are date ranges (often with an identity appended for uniqueness).
- **GUID pattern** — `uniqueidentifier` (16 bytes). Unique and static but wide and, with `NEWID()`, random → inserts scatter across the index causing splits and fragmentation. `NEWSEQUENTIALID()` is increasing only per machine and resets after a restart. Use as a last resort; storing a GUID as `varchar(36)` is worse still.

**Primary key vs clustered index.** They are independent. `PRIMARY KEY` defaults to `CLUSTERED` only when the table has no clustered index yet. Create the clustered index first (or write `PRIMARY KEY NONCLUSTERED`) when you want them on different columns — e.g. a GUID primary key for the application, clustered on an identity column.

### 4. Worked example
```sql
-- Same data, three clustering choices
CREATE TABLE dbo.C_Identity (ID INT IDENTITY CONSTRAINT PK_CI PRIMARY KEY CLUSTERED,
                             G UNIQUEIDENTIFIER DEFAULT NEWID(), Payload CHAR(200) DEFAULT 'x');
CREATE TABLE dbo.C_Guid     (G UNIQUEIDENTIFIER DEFAULT NEWID() CONSTRAINT PK_CG PRIMARY KEY CLUSTERED,
                             ID INT IDENTITY, Payload CHAR(200) DEFAULT 'x');
CREATE TABLE dbo.C_GuidPK_IdCl (ID INT IDENTITY, G UNIQUEIDENTIFIER DEFAULT NEWID(),
                             Payload CHAR(200) DEFAULT 'x');
CREATE UNIQUE CLUSTERED INDEX CX_IdCl ON dbo.C_GuidPK_IdCl (ID);
ALTER TABLE dbo.C_GuidPK_IdCl ADD CONSTRAINT PK_IdCl PRIMARY KEY NONCLUSTERED (G);

-- insert 200,000 default rows into each (loop or INSERT … SELECT TOP (200000) … FROM a numbers source)

SELECT OBJECT_NAME(ps.object_id) AS tbl, i.name, ps.index_level, ps.page_count,
       ps.avg_fragmentation_in_percent, ps.avg_page_space_used_in_percent
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED') ps
JOIN sys.indexes i ON i.object_id = ps.object_id AND i.index_id = ps.index_id
WHERE OBJECT_NAME(ps.object_id) LIKE 'C[_]%' AND ps.index_level = 0;

SELECT name, type_desc, is_primary_key, is_unique
FROM sys.indexes WHERE object_id = OBJECT_ID('dbo.C_GuidPK_IdCl');
```
**What you will see:**
- `C_Identity`: fragmentation near 0%, pages ~99% full — every insert appended to the last page.
- `C_Guid`: fragmentation typically > 95% and pages ~65–75% full — random inserts split pages throughout the index, so it needs roughly a third more pages for the same rows.
- `C_GuidPK_IdCl`: the clustered index looks like `C_Identity`; only the nonclustered PK on G fragments, and that index is narrow (16-byte key + 4-byte locator) so the damage is contained.
- The `sys.indexes` query shows `CX_IdCl` as CLUSTERED and `PK_IdCl` as NONCLUSTERED with `is_primary_key = 1` — proof that "primary key = clustered" is only a default.

### 5. Under the hood
- **Uniquifier details.** In a non-unique clustered index, the first row with a given key gets no uniquifier; duplicates get an incrementing 4-byte value stored as a variable-length column (so it also brings the variable-length overhead if the row had none). It appears in nonclustered indexes too, as part of the locator. It is per key value, so you run out only after ~2.1 billion duplicates of the same key.
- **Non-leaf rows** contain only the clustering key and a 6-byte child page pointer, which is why a narrow key gives a high fan-out (hundreds of entries per page) and a shallow tree.
- **Hot spot at the end.** An ever-increasing key concentrates all inserts on the last leaf page. At very high concurrency this produces `PAGELATCH_EX` contention ("last page insert contention"). SQL Server 2019 added `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` to mitigate it; other options are hash-partitioning or a different key.
- **Rebuilding a clustered index** doesn't change row locators (the key stays the same), so NC indexes are untouched. Converting heap ↔ clustered, or dropping and re-creating the clustered index, changes every locator, so every nonclustered index is rebuilt — twice if you drop and then create. `CREATE … WITH (DROP_EXISTING = ON)` does it in one step and rebuilds NC indexes once (or not at all if the key definition doesn't change).

### 6. Performance and design implications
- Singleton lookups cost one read per level; range reads add one read per leaf page of qualifying rows.
- A wide clustering key inflates every NC index: 10 NC indexes × 1 billion rows × 12 extra bytes ≈ 120 GB.
- Random keys cause splits, extra log, lower page density (more pages read for the same data) and more memory per row cached.
- Choose the key that serves the dominant access path; often that's the PK, but for tables mostly read by a date range or a parent ID, clustering there can remove many lookups.
- Analytical tables with mostly scans may be better as clustered columnstore instead.
- Measure: `sys.dm_db_index_physical_stats` (density, fragmentation, depth), `sys.dm_db_index_operational_stats` (`leaf_allocation_count` for splits), the `Page Splits/sec` counter, `PAGELATCH_EX` waits on the last page.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `PRIMARY KEY [CLUSTERED|NONCLUSTERED]` | Creates the PK with chosen index type | Separate PK from clustering key |
| `CREATE UNIQUE CLUSTERED INDEX … ON t (key)` | Clustered index without a constraint | Clustering on a non-PK column |
| `WITH (DROP_EXISTING = ON)` | Redefines an index in one atomic step | Changing clustered key or options |
| `WITH (OPTIMIZE_FOR_SEQUENTIAL_KEY = ON)` | Reduces last-page insert contention (2019+) | Very high-rate inserts on identity keys |
| `NEWSEQUENTIALID()` (column default only) | Increasing GUIDs per machine since restart | When GUID keys are required |
| `CREATE SEQUENCE` / `sp_sequence_get_range` | App-side pre-allocated integer keys | Replacing GUIDs generated in the app |
| `sys.dm_db_index_physical_stats` | Depth, page count, fragmentation, density | Comparing key choices |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Identity/sequence key for typical OLTP tables | Extreme insert rates → last-page latch contention (use `OPTIMIZE_FOR_SEQUENTIAL_KEY` or partition) |
| Parent-child composite key when children are mostly read by parent | Children are frequently added to old parents (mid-index inserts) |
| Date key for append-only, range-queried event tables | Dates aren't unique — add a tiebreaker column, or accept the uniquifier |
| Short natural code for small lookup tables | The "natural" value can change (e.g. email addresses) |
| GUID only when distributed generation is mandatory | High insert rates and many NC indexes — prefer GUID as NC PK + identity as clustered |

### 9. Common mistakes
- Assuming the PK is always clustered → a table built with a clustered index first gets a nonclustered PK → check `sys.indexes.type_desc`.
- Clustering on `NEWID()` GUIDs → constant splits, 30%+ wasted space, heavy fragmentation → identity clustering + nonclustered GUID PK.
- Clustering on a frequently updated column → each update moves the row and rewrites every NC index → choose a static key.
- Non-unique clustered index on a low-selectivity column → uniquifiers everywhere and bigger NC indexes → make it unique by appending an identity.
- Dropping and re-creating a clustered index to "change" it → NC indexes rebuilt twice, constraints/FKs break → `DROP_EXISTING = ON`.
- Believing rows are stored "physically sorted on disk" → order is logical via page links; fragmentation is exactly the gap between the two.

### 10. 🎯 Interview kit
**30-second answer:** "A clustered index is the table itself stored as a B-tree ordered by the clustering key; the leaf level holds the full rows, so there's only one per table. Every nonclustered index stores the clustering key as its row locator, so the ideal key is narrow, unique, static and ever-increasing — typically an INT/BIGINT identity. Random GUIDs cause page splits and fragmentation, and the PK is only clustered by default, so you can separate them."

**Structure a longer answer:** what it is (data in a B-tree) → why the key matters (locator in all NC indexes, insert pattern) → four properties → patterns (identity, parent-child, date, GUID) → trade-offs (last-page contention, range queries) → scenario: GUID-clustered table with heavy fragmentation fixed by an identity clustering key and a nonclustered GUID PK.

**Likely follow-ups:**
- *What happens with a non-unique clustered index?* → 4-byte uniquifier on duplicates, also carried in NC indexes.
- *Why can't a table have two clustered indexes?* → rows can be stored in only one order.
- *Is an identity key always best?* → mostly; watch last-page latch contention at very high concurrency, and range queries on other columns.
- *How do you change the clustering key on a big table?* → `CREATE … WITH (DROP_EXISTING = ON, ONLINE = ON)`, plan for log and time; FKs/PK need constraint handling.

**Signals interviewers look for:** knows the clustering key propagates to NC indexes; can justify the four properties; distinguishes PK from clustered index; aware of GUID fragmentation and last-page contention.

### 11. Check yourself
- [ ] Draw a 3-level clustered index and trace a seek and a range scan.
- [ ] Explain each of narrow/unique/static/ever-increasing with its concrete cost when violated.
- [ ] Reproduce the identity vs GUID fragmentation difference and read `avg_page_space_used_in_percent`.
- [ ] Create a table with a nonclustered PK and a clustered index on another column.
- [ ] Explain when a parent-child clustered key beats an identity key.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Foreign Key
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 9 Index Architecture
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Clustered Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 2 Primary Keys Are Always Clustered › Myth 2: Primary Keys Are Always Clustered
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Globally Unique Identifier
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Unique Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Drop and Create
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Hash and Range Indexes › Unique Index
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Syntax › Key Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Surrogate Key

