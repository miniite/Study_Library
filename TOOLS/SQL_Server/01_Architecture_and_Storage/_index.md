---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage"
level: 3
type: "curriculum"
curriculum: "sql_server"
---

# Architecture and Storage

**L3 Practitioner** · [↑ SQL Server](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL Server is a set of cooperating engine layers that turn a T-SQL statement into reads and writes of 8 KB pages, cached in memory and protected by a sequential transaction log.

### 1. The big picture
Almost every performance or reliability question about SQL Server ends up at the storage layer: *how many pages did this query touch, were they in memory, and how much log did the change generate?* This block of the curriculum builds that mental model from the bottom up — pages and extents, how rows are laid out inside a page, how pages are organised into heaps and B-trees, how compression changes the layout, how the transaction log makes changes durable, and how tempdb and the version store support everything else.

The parent idea to hold on to is this: **a query is always answered from memory, never directly from disk.** If the pages a query needs are not in the *buffer cache* (also called the buffer pool), the engine first reads them from the data files into memory and then serves the query from the cached copy. So every row you return that you did not need pushes other useful pages out of cache, and a later query has to go back to disk. Once you see the engine this way, advice such as "select only the columns you need", "index for your predicates" and "keep transactions short" stops being folklore and becomes arithmetic.

### 2. Core concepts and vocabulary
- **Instance** — one installed copy of the SQL Server service (one `sqlservr.exe` process) with its own memory, configuration and set of databases.
- **Database** — a container of objects (tables, views, stored procedures, functions, indexes, constraints, triggers, types) plus the files that hold them. One application usually has one database, but an ERP may put everything in one, and one application can use several.
- **Data file (.mdf / .ndf)** — the file(s) that store pages of tables and indexes. A database can have several data files, grouped into **filegroups**.
- **Log file (.ldf)** — the transaction log: a sequential record of every change, used for rollback, crash recovery and log backups.
- **Page** — the 8 KB unit of storage and I/O. Everything (data, index, allocation maps) lives on pages.
- **Extent** — eight contiguous pages (64 KB), the unit in which space is allocated.
- **Buffer pool / buffer cache** — the memory area that holds copies of data pages. Reads and writes happen here first.
- **Heap** — a table without a clustered index; rows are stored in no particular order.
- **B-tree (clustered / nonclustered index)** — a balanced tree of pages that keeps rows (or index keys) sorted and allows seeks.
- **tempdb** — a shared system database for temporary objects, spills, and the row version store.
- **FILESTREAM** — an option that stores large `VARBINARY(MAX)` values as files in the file system while keeping them transactionally part of the database.

### 3. How it works, step by step
What happens when you run `UPDATE dbo.Orders SET Status = 'Shipped' WHERE OrderId = 42;`:

1. **Protocol layer** receives the request from the client (TDS over TCP or shared memory).
2. **Relational engine (query processor)** parses the text, binds names to objects, and the **optimizer** chooses a plan — say, a seek on the clustered index of `Orders`.
3. **Storage engine** executes the plan's data access. The **access methods** walk the B-tree from root to leaf to find the page that holds `OrderId = 42`.
4. **Buffer manager** checks whether that page is in the buffer pool. If not, it issues a read from the data file and waits (`PAGEIOLATCH_*` waits) until the page is in memory.
5. **Lock manager** takes an exclusive lock on the row (and intent locks on the page and table) so other transactions cannot change it concurrently.
6. The change is made **to the page in memory**; the page is now *dirty*.
7. A **log record** describing the change is written to the log buffer. At commit, the log buffer is flushed to the log file on disk (`WRITELOG` waits). This is *write-ahead logging*: the log reaches disk before the data page does.
8. Later, a **checkpoint** or the **lazy writer** writes the dirty page to the data file. If the server crashed before that, recovery replays the log.

| Layer | Main job | You see it in |
|---|---|---|
| Protocol | Network, sessions | `sys.dm_exec_connections` |
| Relational engine | Parse, optimize, execute plans | Execution plans, `sys.dm_exec_query_stats` |
| Storage engine | Access methods, locking, logging, buffer management | `sys.dm_os_buffer_descriptors`, `sys.dm_tran_locks`, `sys.dm_db_log_space_usage` |
| SQLOS | Scheduling, memory, waits | `sys.dm_os_wait_stats`, `sys.dm_os_schedulers` |

### 4. Worked example
```sql
-- 1. Look at a database's files
SELECT name, type_desc, physical_name, size * 8 / 1024 AS size_mb, growth
FROM sys.database_files;

-- 2. How much of the buffer pool each database is using
SELECT DB_NAME(database_id) AS db, COUNT(*) * 8 / 1024 AS cached_mb
FROM sys.dm_os_buffer_descriptors
GROUP BY database_id
ORDER BY cached_mb DESC;

-- 3. See logical vs physical reads for a query
SET STATISTICS IO ON;
SELECT * FROM Sales.SalesOrderDetail;              -- all columns
SELECT SalesOrderID, ProductID FROM Sales.SalesOrderDetail; -- only what we need
SET STATISTICS IO OFF;
```
Reading the output:
- Query 1 shows one `ROWS` file per data file and one `LOG` file. Multiple data files are normal; multiple log files usually are not (see §9).
- Query 2 tells you which database "owns" memory. A reporting database that dominates the cache can explain why OLTP queries suddenly do physical reads.
- In query 3, the `Messages` tab shows `logical reads` (pages read from cache) and `physical reads` / `read-ahead reads` (pages fetched from disk). The narrow query may read far fewer pages if a narrower nonclustered index covers it — fewer pages, less cache pressure.

### 5. Under the hood
- **SQLOS** is a user-mode "operating system" inside SQL Server. It runs one scheduler per CPU core and uses cooperative (non-preemptive) scheduling: a worker runs until it must wait, then records a *wait type*. That is why wait statistics are the primary diagnostic tool.
- **Write-ahead logging (WAL)** is the key durability rule: a dirty data page may not be written to disk until all log records describing changes to it are on disk. Commits wait only for the sequential log write, not for random data-page writes — which is why log latency matters so much.
- **Log is sequential, data is random.** Log records are always appended in order; that is why splitting the log into several files gives no performance benefit — SQL Server fills the first file before using the next.
- **Multiple data files** do help: they spread allocation work across more allocation pages (less contention, especially in tempdb), allow parallel I/O, and make migration and piecemeal restore easier.
- **FILESTREAM** keeps large binary objects on the NTFS file system for efficient streaming, but integrates them with database security, transactions and backups. Recommended for files above about 1 MB; Windows only.

### 6. Performance and design implications
- **Memory is the first line of performance.** Every unnecessary column or row read evicts something else. Measure with `SET STATISTICS IO` and `sys.dm_os_buffer_descriptors`.
- **Log latency is commit latency.** Watch `WRITELOG` waits; put the log on low-latency storage; avoid many tiny transactions in loops (each commit waits for a log flush).
- **Data I/O latency** shows up as `PAGEIOLATCH_SH/EX` waits; fix by reducing reads (indexes, narrower queries) before buying faster disks.
- **Instant file initialization** lets data files grow without zeroing; log files always need zeroing (except small growths in recent versions), so big log growths and restores that must recreate the log take time.
- **Cloud variants** (Azure SQL, Synapse) separate compute and storage, and features such as PolyBase/external tables let you query files in a data lake as if they were tables — the same engine ideas apply, but storage is remote.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.database_files` | Lists data/log files, sizes, growth | Capacity checks |
| `SET STATISTICS IO ON` | Shows logical/physical reads per table | Tuning any query |
| `sys.dm_os_buffer_descriptors` | Pages in cache by database/object | Memory pressure analysis |
| `sys.dm_os_wait_stats` | Cumulative waits | First step of any performance triage |
| `sys.dm_db_log_space_usage` | Log size and percent used | Log growth investigations |
| `ALTER DATABASE … ADD FILE` | Adds a data (or log) file | Spread I/O; emergency log space |
| `CHECKPOINT` | Flushes dirty pages of current DB | Testing, before demos of recovery |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Several data files in a filegroup — high allocation activity, very large DBs, tempdb | Files of unequal size (proportional fill becomes unbalanced) |
| One log file, sized correctly up front | Adding log files for "performance" — no gain, slower restores |
| FILESTREAM for large documents/images | Small blobs (<1 MB) — `VARBINARY(MAX)` in-row/LOB is simpler; Linux deployments |
| External tables / PolyBase for occasional lake queries | Hot OLTP paths that need indexes and low latency |

### 9. Common mistakes
- **Returning `SELECT *` "just in case"** → buffer cache fills with unneeded data, older pages are evicted, more physical reads → select only needed columns and rows.
- **Adding a second log file to fix `WRITELOG` waits** → all writes still go to the first file; restores must zero two files → fix the real cause (faster log storage, larger batches, minimal logging for bulk loads); use a second log file only as a temporary emergency when the log volume is full, then remove it.
- **Assuming committed data is already in the data file** → confusion about recovery and checkpoints → remember WAL: durability comes from the log.
- **Ignoring tempdb** → spills, version store growth and allocation contention hurt every database on the instance → size and configure tempdb deliberately.

### 10. 🎯 Interview kit
**30-second answer:** SQL Server has a relational engine that parses and optimizes queries, and a storage engine that reads and writes 8 KB pages through the buffer pool. Queries are always satisfied from memory; missing pages are read in first. Changes are made in memory and recorded in a sequential transaction log that is flushed at commit (write-ahead logging); dirty pages are written to data files later by checkpoint and the lazy writer.

**Structure a longer answer:** clarify (on-prem vs Azure; OLTP vs analytics) → walk one query through the layers → explain pages, buffer pool, log and WAL → trade-offs (memory vs I/O, log latency) → close with a scenario such as diagnosing `PAGEIOLATCH` vs `WRITELOG` waits.

**Likely follow-ups:**
- *Why don't multiple log files help?* → log writes are sequential; the second file is used only when the first is full.
- *What happens on crash?* → recovery redoes committed changes from the log and undoes uncommitted ones.
- *What is in tempdb?* → temp tables, spills, row versions for snapshot isolation, internal work tables.

**Signals interviewers look for:** you know the page/buffer/log triangle, you reach for wait stats and `STATISTICS IO`, and you don't recommend "add a log file" or "add hardware" before measuring.

### 11. Check yourself
- [ ] Draw the path of an `UPDATE` from client to log file to data file.
- [ ] Explain why a query never reads directly from disk.
- [ ] Run `SET STATISTICS IO ON` on two versions of a query and explain the difference in logical reads.
- [ ] Explain when a second data file helps and why a second log file does not.
- [ ] Name the wait types that point to data I/O and to log I/O.
<!-- explainer:end -->

## In this section (study in order)

1. [Engine Architecture](<Engine_Architecture/_index.md>) (L4 Advanced)
2. [Pages and Extents](<Pages_and_Extents/_index.md>) (L4 Advanced)
3. [Heaps and Forwarded Records](<Heaps_and_Forwarded_Records/_index.md>) (L4 Advanced)
4. [B-Tree Structure](<B-Tree_Structure/_index.md>) (L4 Advanced)
5. [Row Storage and Data Types](<Row_Storage_and_Data_Types/_index.md>) (L4 Advanced)
6. [Data Compression](<Data_Compression/_index.md>) (L4 Advanced)
7. [Transaction Log and Recovery](<Transaction_Log_and_Recovery/_index.md>) (L4 Advanced)
8. [tempdb and the Version Store](<tempdb_and_Version_Store/_index.md>) (L4 Advanced)

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Data Architecture in Synapse
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.fulltextindexcolumns › Storage Basics
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Getting started
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Hybrid Cloud Architectures
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Instance and database management › 9.4 #46 Using multiple log files
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Introducing SQL Server › 1.2.1 Overview of the Database Engine
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Exploring Database Concepts › Database as Container
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › External Tables
- *Practical Graph Structures in SQL Server and Azure SQL* — Louis Davidson — Data Structures and Algorithms › 2. Data Structures and Algorithms
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 12: Azure SQL and Synapse Integration › Advantages of PolyBase and External Tables

