---
id: "TOOLS/SQL_Server/02_Indexing/Memory_Optimized_Indexes"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# In-Memory OLTP and Hash Indexes

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Memory-optimized tables (In-Memory OLTP) use memory-only indexes built from pointers rather than pages: **hash indexes** for equality lookups and **range (Bw-tree) nonclustered indexes** for ranges and ordering. They need no latches or locks, and they are rebuilt from data at every startup.

### 1. The big picture
Everything in this section so far lives on 8 KB pages that the buffer pool reads from disk, protected by latches and locks. In-Memory OLTP (introduced in SQL Server 2014 under the codename *Hekaton*) throws that model away for selected tables. A **memory-optimized table** lives entirely in memory. Rows are linked by memory pointers, concurrency uses **multi-version optimistic** control (no locks, no latches), and durability comes from the transaction log plus checkpoint files, not from data pages.

Indexes on these tables are fundamentally different. They are never written to disk, never logged, and exist only as pointer structures in memory, rebuilt when the database starts. A table needs at least one index, because indexes are what *connect* the rows. There are two kinds. **Hash indexes** map a key's hash to a bucket of pointers, which is ideal for `WHERE id = @x`. **Nonclustered range indexes** are lock-free B-tree variants (Bw-trees) that support ranges and ORDER BY. A columnstore index can be added too, for analytics. Choosing between hash and range, and sizing the hash bucket count, are the key design decisions.

### 2. Core concepts and vocabulary
- **Memory-optimized table**: `WITH (MEMORY_OPTIMIZED = ON)`. Rows live in memory and require a `MEMORY_OPTIMIZED_DATA` filegroup.
- **Durability**: `SCHEMA_AND_DATA` (fully durable, the default) or `SCHEMA_ONLY` (data lost on restart; good for staging/session state).
- **Row versions**: an update creates a new row version with begin/end timestamps. Readers see the version valid at their start time. Garbage collection removes old versions.
- **Hash index**: an array of *buckets*. Each bucket holds a pointer to a chain of rows whose key hashes to it. Supports only equality on *all* key columns.
- **BUCKET_COUNT**: the number of buckets, rounded up to a power of two. Fixed until the index is rebuilt.
- **Hash chain / collision**: several distinct keys in one bucket (collision), or many rows with the same key (duplicates), make chains long, and every lookup walks the chain.
- **Range (nonclustered) index**: a Bw-tree with variable-size, lock-free pages and a *mapping table* of logical page IDs. Supports seeks, ranges, ordering and inequality.
- **Natively compiled stored procedure**: a T-SQL procedure compiled to machine code, which can access only memory-optimized tables. It removes interpretation overhead.
- **Interop**: ordinary (interpreted) T-SQL accessing memory-optimized tables.

### 3. How it works, step by step
**Hash index lookup:**
1. Compute `hash(key)` → bucket number.
2. Follow the bucket's pointer to the first row in the chain.
3. Walk the chain via each row's per-index *next* pointer, comparing keys and checking version timestamps for visibility.
4. Return matching visible rows.

A hash index **cannot** seek on a prefix of a multi-column key or on a range. With a key of `(CustomerID, OrderDate)`, the query `WHERE CustomerID = 5` gets a full index scan, because the hash is computed over both columns.

**Range index lookup:**
1. Start at the root of the Bw-tree. Pages are logical IDs resolved through the mapping table to memory addresses.
2. Navigate to the leaf by key comparison, as in a B-tree.
3. Leaf pages hold key values and row pointers. Follow them to rows. Leaf pages are linked in one direction, so a range index scans efficiently in its declared order only. A reverse-order request requires a sort.
4. Updates to pages are made by prepending *delta records* and swapping the mapping-table pointer atomically (compare-and-swap). No latches are needed.

**Choosing:**

| Query pattern | Index type |
|---|---|
| `WHERE key = @v` on the full key, high-cardinality key (PK, session ID) | Hash |
| Ranges `>`, `<`, `BETWEEN`, `ORDER BY`, prefix of a multi-column key | Range (nonclustered) |
| Many duplicate values (status, type) | Range (hash chains would be long) |
| Large analytic scans | Clustered columnstore on the memory-optimized table |
| Unsure / mixed | Range: performs reasonably for everything; hash is a specialisation |

**Sizing BUCKET_COUNT:**
1. Estimate the number of *distinct* key values the table will hold, including growth.
2. Set `BUCKET_COUNT` to roughly 1–2× that number. SQL Server rounds it up to the next power of two.
3. Too low → long chains → slow lookups and inserts. Too high → wasted memory (8 bytes per bucket) and slower full scans of the index, though lookups stay fast. Erring high is safer than erring low.

### 4. Worked example
```sql
-- Prerequisite: a memory-optimized filegroup
ALTER DATABASE Shop ADD FILEGROUP Shop_mod CONTAINS MEMORY_OPTIMIZED_DATA;
ALTER DATABASE Shop ADD FILE (NAME = 'Shop_mod', FILENAME = 'D:\Data\Shop_mod') TO FILEGROUP Shop_mod;

CREATE TABLE dbo.ShoppingCart
(
    CartID      bigint      NOT NULL
        PRIMARY KEY NONCLUSTERED HASH WITH (BUCKET_COUNT = 2000000),   -- ~1M active carts, x2
    UserID      int         NOT NULL,
    CreatedAt   datetime2   NOT NULL,
    TotalAmount decimal(10,2) NOT NULL,
    INDEX IX_User_Created NONCLUSTERED (UserID, CreatedAt)              -- range index
)
WITH (MEMORY_OPTIMIZED = ON, DURABILITY = SCHEMA_AND_DATA);

-- Point lookup → hash index seek
SELECT * FROM dbo.ShoppingCart WHERE CartID = 123456;
-- Range / prefix → range index seek
SELECT * FROM dbo.ShoppingCart WHERE UserID = 42 AND CreatedAt >= '2026-01-01';

-- Check hash index health
SELECT OBJECT_NAME(hs.object_id) AS tbl, i.name,
       hs.total_bucket_count, hs.empty_bucket_count,
       hs.avg_chain_length, hs.max_chain_length
FROM sys.dm_db_xtp_hash_index_stats AS hs
JOIN sys.indexes AS i ON i.object_id = hs.object_id AND i.index_id = hs.index_id;

-- Resize buckets (rebuilds the index, offline for the table)
ALTER TABLE dbo.ShoppingCart
    ALTER INDEX PK__Shopping__... REBUILD WITH (BUCKET_COUNT = 4194304);
```
Reading `sys.dm_db_xtp_hash_index_stats`:
- `empty_bucket_count` above ~33% of total, with `avg_chain_length` ≈ 1 → healthy, or slightly oversized.
- `avg_chain_length` > ~10 with few empty buckets → too few buckets: rebuild with a larger count.
- `avg_chain_length` high *and* many empty buckets → many duplicate key values. A hash index is the wrong choice; use a range index.

### 5. Under the hood
- **Rows carry the index pointers.** Each row header has one *next* pointer per index, so a table's index count is fixed in its row layout. Early versions allowed at most 8 indexes and no `ALTER TABLE`. SQL Server 2016 added `ALTER TABLE` (it rebuilds the table in memory), and 2017 removed the 8-index limit.
- **No index I/O, no logging.** Only data rows are logged (and more compactly than disk tables). After a restart, SQL Server loads rows from checkpoint file pairs plus the log tail and **rebuilds every index**. Recovery time therefore grows with data size and index count.
- **Statistics.** Memory-optimized tables have statistics. Auto-update has been supported since 2016. Natively compiled procedures get their plans at compile time, so recompile them (`sp_recompile`) after large data changes.
- **Optimistic concurrency.** No locks are taken. Conflicting writes fail fast: error 41302 for an update conflict, and 41305/41325 for validation failures under REPEATABLE READ/SERIALIZABLE. Applications need retry logic.
- **Garbage collection.** Old row versions stay reachable through the index chains until GC unlinks them. Long-running transactions delay GC and lengthen chains.
- **Memory sizing.** Plan for data plus versions plus indexes: roughly hash buckets × 8 bytes + range index pages + row versions. Use Resource Governor to bind the database to a pool, and monitor with `sys.dm_db_xtp_table_memory_stats`.
- **Columnstore on memory-optimized tables.** A clustered columnstore index can be added for analytics. It lives on disk-like storage, but it is kept in sync with the in-memory rows.

### 6. Performance and design implications
- In-Memory OLTP shines for latch/lock contention hotspots: high-concurrency inserts (sequential keys), session state, staging/ETL tables (`SCHEMA_ONLY`), and table-variable replacements (memory-optimized table types).
- Gains are largest when combined with natively compiled procedures and short transactions. Interop access still removes latching but keeps interpretation overhead.
- Watch total memory, recovery time, chain lengths, and conflict/retry rates.
- Not everything should move: large, rarely-hot tables waste memory, and some features (e.g. some data types, cross-database queries, certain constraints in older versions) are limited.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `WITH (MEMORY_OPTIMIZED = ON, DURABILITY = …)` | Creates a memory-optimized table | Contention hotspots, staging |
| `… NONCLUSTERED HASH WITH (BUCKET_COUNT = n)` | Hash index | Equality lookups on the full key |
| `INDEX ix NONCLUSTERED (cols)` | Range (Bw-tree) index | Ranges, ordering, duplicates |
| `sys.dm_db_xtp_hash_index_stats` | Buckets, empty buckets, chain lengths | Sizing check |
| `sys.dm_db_xtp_index_stats` | Scans, rows touched per index | Usage analysis |
| `sys.dm_db_xtp_table_memory_stats` | Memory per table/index | Capacity planning |
| `ALTER TABLE … ALTER INDEX … REBUILD WITH (BUCKET_COUNT = n)` | Resize a hash index | Chains too long |
| `CREATE PROCEDURE … WITH NATIVE_COMPILATION, SCHEMABINDING` | Natively compiled proc | Hot OLTP code paths |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Hash: unique or high-cardinality key, equality only | Range queries, prefix seeks, many duplicates |
| Range index: general-purpose default | Pure point lookups at extreme rates (hash is faster) |
| `SCHEMA_ONLY`: staging, temp, session data | Data must survive a restart |
| In-Memory OLTP at all: latch/lock contention, latency-critical OLTP | Large cold tables, limited RAM, long-running transactions |

### 9. Common mistakes
- A hash index on a low-cardinality column → huge chains → use a range index.
- Under-sizing `BUCKET_COUNT` for future growth → chains grow as data grows → size for 1–2× the expected distinct keys and monitor.
- Expecting a seek on the first column of a multi-column hash key → full scan → a range index, or a hash on exactly the queried columns.
- No retry logic for write conflicts → user-facing errors → catch 41302/41305/41325/41301 and retry.
- Ignoring recovery time → slow database startup after restart → keep indexes lean, and size the memory and storage for the checkpoint files.

### 10. 🎯 Interview kit
**30-second answer:** "Memory-optimized tables keep rows in memory with pointer-based, latch-free indexes that aren't logged and are rebuilt at startup. Hash indexes are bucket arrays for equality on the full key; size BUCKET_COUNT at 1–2× the distinct keys. Range indexes are Bw-trees for ranges, ordering and duplicates. Concurrency is optimistic and multi-version, so you need retry logic."
**Structure a longer answer:** what In-Memory OLTP changes (no pages, latches or locks) → hash vs range mechanics → bucket sizing and chain diagnostics → durability options and recovery → when to use it.
**Likely follow-ups:** *Why can't a hash index do a range?* → the hash destroys ordering. *What does a long chain mean?* → too few buckets or many duplicates (check empty buckets). *Are indexes logged?* → no; they are rebuilt on recovery.
**Signals interviewers look for:** correct hash limitations, sizing method, awareness of conflicts and retries, and not overselling In-Memory OLTP.

### 11. Check yourself
- [ ] Choose hash or range for five sample queries, with reasons.
- [ ] Compute a BUCKET_COUNT for 3M expected distinct keys (answer: round up to 4,194,304 or 8,388,608).
- [ ] Diagnose three `sys.dm_db_xtp_hash_index_stats` outputs.
- [ ] Explain why memory-optimized indexes aren't logged and what that means at startup.
- [ ] Write retry logic for an update conflict.
<!-- explainer:end -->

## References

- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 19 Memory-Optimized Oltp Tables And Procedures
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full-Text Search Index Catalog Views and Properties › Hash Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Hash and Range Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Dropping an Index › sys.hashindexes
- *SQL Server 2025 Query Performance Tuning (7th ed.)* — Grant Fritchey — FULL_BOOK_NEEDS_MANUAL_SPLIT (cont.) › Chapter 18 Row-By-Row Processing From Cursors And Other Causes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full-Text Search Index Catalog Views and Properties › Memory-Optimized Tables Overview
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — IntroduCtIon to analytIC data In a transaCtIonal database › OLAP Data in an OLTP Table in a Transactional Database
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — � D › � M
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full-Text Search Index Catalog Views and Properties › Range Indexes
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Tree Structures, Algorithms, and Performance › Kimball Helper Table

