---
id: "01_Databases_and_SQL/Query_Performance/Indexing/Random_Keys_and_Index_Locality"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/Query_Performance/Indexing"
tools: ["AWS", "Snowflake", "MySQL"]
---

# Random Keys and Index Locality

**L4 Advanced** · [↑ Indexing](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Where a new key lands in an index decides how much that insert costs — sequential keys append to the rightmost page (cache-friendly, no splits, but a possible hot spot), random keys land anywhere (page splits, fragmentation, cache misses), and the clustered key is copied into every secondary index, so its size and randomness spread everywhere.

### 1. The big picture
A B-tree index is kept in key order. Every insert must put the new entry in the one correct leaf page. If the key values arrive in increasing order (identity, sequence, timestamp, UUIDv7/ULID), inserts always go to the *last* page: that page is hot in memory, rarely splits mid-way, and pages are filled nearly full. If keys arrive in random order (UUIDv4, hashes, natural keys like SSN or email), each insert lands in a different place in the tree, so the engine must read that page (often from disk), may need to split it, and leaves behind half-empty pages.

This is called **index locality**: do consecutive inserts (and consecutive reads) touch the same or neighbouring pages? Good locality means fewer I/Os, a smaller index and a hotter cache. The idea reaches far beyond SQL Server: InnoDB's clustered primary key, Redshift sort/dist keys, Snowflake clustering keys and DynamoDB partition keys are all "what physical order is the data in, and does the write pattern respect it?" decisions.

### 2. Core concepts and vocabulary
- **Clustered index / clustered primary key** — the table *is* the B-tree ordered by this key (SQL Server clustered index, InnoDB primary key).
- **Secondary (nonclustered) index** — separate B-tree whose leaf entries store the index key plus a pointer to the row: in SQL Server the clustering key (or RID for heaps), in InnoDB the primary key.
- **Index indirection** — secondary indexes reference rows by primary key rather than physical address, making relocations cheap but reads need a second lookup.
- **Page split** — when a full leaf page needs another row, the engine allocates a new page and moves about half the rows; logged, costly, and leaves free space.
- **Fragmentation** — logical order differs from physical order, and/or pages are partially full.
- **Fill factor** — percentage of each leaf page filled on build/rebuild, leaving room for growth.
- **Monotonic / ever-increasing key** — each new value is larger than all previous (identity, sequence, `SYSUTCDATETIME()`, UUIDv7).
- **Hot spot / last-page insert contention** — many concurrent inserts fighting for latches on the final page.
- **Surrogate key** — system-generated artificial identifier (`INT IDENTITY`); **natural key** — real-world attribute (email, SSN, ISBN).
- **Sort key / distribution key (Redshift)**, **clustering key (Snowflake)** — warehouse analogues that decide physical order/placement.
- **UUID v4 / v7 / ULID** — random 128-bit ID vs time-ordered variants that keep a timestamp prefix.

### 3. How it works, step by step
**Insert into a clustered B-tree**
1. Walk root → intermediate → leaf pages comparing keys to find the target leaf.
2. If the leaf is not in memory, read it from disk (page fault).
3. If there is room, put the row in; otherwise split: allocate a new page, move ~half the rows, update the parent (which may split too).
4. Log all changes; the modified pages become dirty and must eventually be written.

**Sequential key** → step 1 always ends at the same right-most leaf, step 2 is a cache hit, step 3 just allocates a *new* page when the last is full (no row movement), leaving earlier pages ~100% full.

**Random key** → step 1 ends at a random leaf out of millions: step 2 is a cache miss once the index exceeds memory, step 3 splits a page that's already full, producing two ~50% pages. Over time pages hover around 70% full and the index is ~40–50% bigger.

**Secondary indexes** pay the same tax and inherit the clustering key: with a 16-byte random GUID cluster key, every nonclustered index entry carries 16 bytes instead of 4, so *all* of them are larger and less cache-friendly.

| Property | Identity/sequence | UUIDv4 | UUIDv7/ULID |
|---|---|---|---|
| Insert target | last page | random page | last page (roughly) |
| Page splits | rare | frequent | rare |
| Fill density | ~100% | ~70% | ~100% |
| Key size | 4–8 bytes | 16 bytes | 16 bytes |
| Guessable | yes | no | partly (time) |
| Generation across nodes | needs coordination | trivial | trivial |

### 4. Worked example
```sql
CREATE TABLE dbo.EvtSeq  (Id BIGINT IDENTITY PRIMARY KEY CLUSTERED, Payload CHAR(200) NOT NULL DEFAULT 'x');
CREATE TABLE dbo.EvtGuid (Id UNIQUEIDENTIFIER DEFAULT NEWID() PRIMARY KEY CLUSTERED, Payload CHAR(200) NOT NULL DEFAULT 'x');
CREATE TABLE dbo.EvtSeqGuid (Id UNIQUEIDENTIFIER DEFAULT NEWSEQUENTIALID() PRIMARY KEY CLUSTERED, Payload CHAR(200) NOT NULL DEFAULT 'x');

-- load 1,000,000 rows into each, then compare
SELECT OBJECT_NAME(ips.object_id) AS tbl,
       ips.avg_fragmentation_in_percent AS frag_pct,
       ips.avg_page_space_used_in_percent AS page_fullness,
       ips.page_count
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'SAMPLED') ips
WHERE ips.index_level = 0 AND OBJECT_NAME(ips.object_id) LIKE 'Evt%';
```
What you should see: `EvtSeq` and `EvtSeqGuid` ≈ 0–1% fragmentation, ~99% page fullness; `EvtGuid` shows ~95–99% fragmentation, ~65–75% fullness and noticeably more pages. Also compare `sys.dm_db_index_operational_stats` for `leaf_allocation_count` (page splits) and the elapsed load time with a buffer pool smaller than the table: the random load slows as the index outgrows memory because each insert needs a physical read.

The opposite problem shows up with a *highly concurrent* insert workload on `EvtSeq`: sessions pile onto the last page and you see `PAGELATCH_EX` waits. SQL Server 2019 offers a mitigation:
```sql
ALTER INDEX PK_EvtSeq ON dbo.EvtSeq SET (OPTIMIZE_FOR_SEQUENTIAL_KEY = ON);
```

### 5. Under the hood
- **Why splits hurt.** A split is a multi-page, fully logged operation (new page allocation, row movement, parent update, possibly cascading). Random inserts into full pages cause one almost per insert once the table is mature; that's extra log, extra I/O, and the new page is generally *not* physically adjacent, so range scans later suffer more random I/O (logical vs physical fragmentation).
- **Cache effect.** For sequential keys the working set is "the right edge" — a few pages. For random keys, the working set is the *entire* index; once it is larger than RAM every insert is a read-modify-write on a cold page.
- **Last page contention.** With sequential keys under high concurrency, many threads need an exclusive latch on the same page. `OPTIMIZE_FOR_SEQUENTIAL_KEY` adds a flow-control mechanism: it inspects threads just before they request the page latch and prioritizes ones likely to finish quickly, reducing the convoy that forms behind a thread stalled by a page split. (It is a tuning aid, not a way to fix random keys.)
- **Indirection trade-off (Database Internals).** Referencing rows directly by file offset saves a lookup on reads but forces pointer rewrites in every secondary index whenever a row moves; referencing by primary key (InnoDB, SQL Server's clustered key) makes writes cheaper but adds a primary-index lookup per secondary-index read. This is why the clustered key's *width* matters so much — it is in every secondary entry.
- **Composite / Foreign-Key pattern.** A clustered key like `(OrderID, LineNumber)` for detail rows is ever-increasing only if details arrive right after the header; late-arriving lines cause mid-index inserts and extra fragmentation. It also doubles key width (8 bytes) in each nonclustered index.
- **Warehouse analogues.** Redshift sort keys decide on-disk order so zone maps (per-block min/max) can skip blocks; new loads append to an *unsorted region*, which erodes the benefit until `VACUUM` re-sorts — conceptually index fragmentation. Compound sort keys help only when filters use the leading column(s); interleaved keys give equal weight but cost more on `VACUUM` and hurt as data grows. Snowflake's clustering key is not an index but a hint for micro-partition layout; clustering on a raw high-cardinality timestamp is expensive, an expression like `DATE_TRUNC('hour', ts)` clusters better. DynamoDB's partition key must spread load; a monotonically increasing partition key creates a hot partition, the same hot-spot idea at cluster scale.
- **Distributed IDs.** Snowflake-style IDs (timestamp | worker | sequence), UUIDv7 and ULID are time-prefixed, so locally they behave like sequential keys while remaining globally unique; but within a *single millisecond* across many writers the order is not strict, giving slight locality loss — usually acceptable.

### 6. Performance and design implications
- Prefer a **narrow, static, unique, ever-increasing** clustered key (the classic rule); `INT`/`BIGINT IDENTITY` meets all four.
- Keep random IDs off the clustered index: use `BIGINT IDENTITY` as the cluster key and a unique **nonclustered** index on the GUID/external ID for lookups.
- If a GUID cluster key is mandated, use `NEWSEQUENTIALID()` (SQL Server, only as a column default) or UUIDv7/ULID generated in the app, and lower the fill factor (e.g. 70–80%) on truly random indexes to absorb splits.
- Monitor: `sys.dm_db_index_physical_stats` (fragmentation, fullness), page-split counters (`Page Splits/sec`), `PAGELATCH_EX` waits, buffer pool pressure.
- Remember key size multiplies: `GUID` (16 bytes) vs `INT` (4) in 5 nonclustered indexes over 500 M rows is gigabytes of extra index and cache.
- For ingestion pipelines loading into MySQL/InnoDB, sort the batch by primary key before bulk insert, or use auto-increment keys, to get append behaviour.
- In warehouses: choose sort/cluster keys from the dominant filter columns, keep loads appended in sort order, and schedule `VACUUM`/reclustering; use `SORTKEY AUTO` when unsure.
- Don't over-correct: random keys are fine for small tables or when the whole index fits in memory.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `NEWSEQUENTIALID()` | Generates increasing GUIDs (per boot/server) | Need GUID cluster key in SQL Server |
| `OPTIMIZE_FOR_SEQUENTIAL_KEY = ON` | Flow control for last-page latch contention (2019+) | `PAGELATCH_EX` at high insert rates on sequential keys |
| `FILLFACTOR = n` | Leaves free space in leaf pages | Random-key or update-heavy indexes |
| `sys.dm_db_index_physical_stats` | Fragmentation, page density | Diagnose split damage |
| `ALTER INDEX … REBUILD / REORGANIZE` | Restores order and density | After heavy splits (don't schedule blindly) |
| `SORTKEY AUTO`, `VACUUM` (Redshift) | Choose and restore sort order | Warehouse tables with append loads |
| `CLUSTER BY (expr)` (Snowflake) | Define clustering | Large tables with consistent range filters |
| UUIDv7 / ULID | Time-ordered unique IDs | Distributed ID generation with locality |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| **Identity/sequence cluster key** — single writer database, OLTP, range scans by recency | Multi-region writers needing independent ID generation; ID enumeration is a security concern |
| **UUIDv7/ULID** — distributed generation with good locality | Strict monotonicity or non-leaky IDs are required (embeds time); mixed generators across services |
| **UUIDv4 as non-clustered unique key** — external, unguessable identifier | Using it as the clustered key of a large table or in many FKs |
| **Natural key as clustered key** — stable, short, genuinely unique (e.g. fixed-width code) | Long, variable-length or changing values (SSN, email) — wide secondary indexes and updates |
| **Lower fill factor** — random inserts into mature index | Append-only index — wastes space |
| **Compound sort key** — filters share leading columns | Query mix filters on varying leading columns (consider interleaved or auto) |

### 9. Common mistakes
- **`NEWID()` as clustered PK on a big table** → fragmentation, 2× page count, cold-cache inserts → cluster on identity, index the GUID separately, or use sequential/v7 IDs.
- **Wide clustered key** → every secondary index inflated → keep it narrow.
- **Rebuilding indexes nightly "to fix fragmentation"** from a random key → re-fragments by morning, burns log → fix the key or set fill factor.
- **Using a timestamp as the only key** → duplicates and ties → add a unique tie-breaker.
- **Ignoring last-page contention on sequential keys at very high rates** → `PAGELATCH_EX` queue → `OPTIMIZE_FOR_SEQUENTIAL_KEY`, hash-partition, or spread writers.
- **Never running `VACUUM` in Redshift** after loads → growing unsorted region, zone maps useless → schedule vacuum/auto-vacuum and load in sort order.
- **Clustering Snowflake on a raw timestamp/UUID** → high reclustering cost, little pruning → cluster on a coarse expression.
- **Assuming secondary indexes are free** → write amplification for each insert/update → index only what queries need.

### 10. 🎯 Interview kit
**30-second answer:** "A B-tree has to insert each key in sorted position. Sequential keys always append to the last page, so the page is hot in cache, there are no mid-index splits and pages are full. Random keys like UUIDv4 land anywhere, causing page splits, ~30% wasted space and cache misses, and since the clustered key is repeated in every secondary index, a wide random key bloats everything. So I'd cluster on a narrow increasing key and keep the GUID as a separate unique index, or use UUIDv7/ULID."

**Structure a longer answer:** clarify (engine, write rate, single vs multi-node generation, external exposure) → mechanism (splits, cache, indirection) → measurement (fragmentation, page splits, waits) → options (identity, sequential GUID, v7, fill factor, separate index) → trade-offs and a real scenario.

**Likely follow-ups:**
- *Why does InnoDB care so much?* → The PK is the clustered index and every secondary index stores the PK, so a random/wide PK amplifies both.
- *Does sequential always win?* → No: last-page latch contention at very high concurrency; guessable IDs; hot partition in distributed stores.
- *How does UUIDv7 differ from v4?* → 48-bit ms timestamp prefix keeps inserts near the right edge, with random suffix for uniqueness.
- *Snowflake/Redshift equivalent?* → Sort/cluster keys + zone maps/micro-partition metadata; same "layout = index" idea, with VACUUM/reclustering as maintenance.
- *DynamoDB GSI vs relational index?* → GSI has its own capacity cost and eventually consistent reads; design keys for even distribution.

**Signals interviewers look for:** you separate *identity of the row* from *physical order*, quantify costs (fragmentation, key width), name the contention counter-case, and propose measurement instead of folklore.

### 11. Check yourself
- [ ] Draw what happens in a B-tree leaf when a random key is inserted into a full page.
- [ ] Explain why a 16-byte clustering key affects the size of unrelated nonclustered indexes.
- [ ] Reproduce the three-table demo and interpret `avg_page_space_used_in_percent`.
- [ ] Say when `OPTIMIZE_FOR_SEQUENTIAL_KEY` helps and when it doesn't.
- [ ] Propose a company-wide key policy covering OLTP, distributed services and warehouses, with exceptions.
<!-- explainer:end -->

## Implementations (tools)

- [AWS](<../../../TOOLS/AWS/_index.md>)
- [Snowflake](<../../../TOOLS/Snowflake/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [What is a natural key versus a surrogate key in a dimension table, and how does the choice affect the primary key index?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20271_What_is_a_natural_key_versus_a_surrogate_key_in_a_dimension_.md>)
- [Deep] [In a CDC-based pipeline, why is it important that the source primary key is indexed for the change-capture reads to be efficient?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20309_In_a_CDC-based_pipeline_why_is_it_important_that_the_source_.md>)
- [Deep] [What is a Redshift sort key and a distribution key, and how do they play the role that indexes play in a row-store?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20331_What_is_a_Redshift_sort_key_and_a_distribution_key_and_how_d.md>)
- [Deep] [How does InnoDB's clustered primary key affect the size and performance of secondary indexes?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19595_How_does_InnoDB_s_clustered_primary_key_affect_the_size_and_.md>)
- [Deep] [How does a Snowflake clustering key differ from a relational index and when is defining one actually worth the reclustering cost?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20370_How_does_a_Snowflake_clustering_key_differ_from_a_relational.md>)
- [Deep] [In Amazon Redshift, how do sort keys (compound vs interleaved) act like a lightweight index, and when does an interleaved sort key hurt?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20394_In_Amazon_Redshift_how_do_sort_keys_compound_vs_interleaved_.md>)
- [Deep] [Explain how vacuum/analyze and unsorted-region growth in Redshift silently erode sort-key benefits, mirroring index bloat in an OLTP database.](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-20453_Explain_how_vacuum_analyze_and_unsorted-region_growth_in_Red.md>)
- [Expert] [How does DynamoDB's Global Secondary Index differ from a relational index in cost and consistency?](<../../../PRACTICE/01_Databases_and_SQL/Query_Performance/Indexing/Q-19638_How_does_DynamoDB_s_Global_Secondary_Index_differ_from_a_rel.md>)

## Related concepts

- [Primary and Unique Keys](<../../Database_Design/Constraints_and_Keys/Primary_and_Unique_Keys.md>) *(Constraints and Keys)*
- [Large Values and TOAST](<../../SQL_Language/SQL_Fundamentals/Large_Values_and_TOAST.md>) *(SQL Fundamentals)*
- [Composite vs Surrogate Keys](<../../Database_Internals/DBMS_Concepts/Composite_vs_Surrogate_Keys.md>) *(DBMS Concepts)*

## References

- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Designing Data Models and Schema › Physical data modeling in Amazon Redshift: Choosing the best sort key
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Modeling Strategies for Amazon DynamoDB › Selecting the right sort key
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — Data Modeling Strategies for Amazon DynamoDB › Utilizing global secondary indexes and local secondary indexes
- *AWS Certified Data Engineer Associate Study Guide* — Sakti Mishra et al. — T › U
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 2 Primary Keys Are Always Clustered › Myth 2: Primary Keys Are Always Clustered
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Myth 9 Every Table Should Have a HeapClustered Index › Use Clustered Indexes on Primary Keys by Default
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Foreign Key
- *Database Internals* — Alex Petrov — Introduction and Overview › Primary Index as an Indirection
- *Database Internals* — Alex Petrov — Introduction and Overview › Local and Remote Execution
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Indexes › 11.8.3 Understanding OPTIMIZEFORSEQUENTIALKEY
- *Data Engineering Design Patterns* — Bartosz Konieczny — Pattern: Local Aggregator › Grouping keys
- *Data Engineering Design Patterns* — Bartosz Konieczny — In-Flight Requests › Examples
- *Hadoop: The Definitive Guide* — Tom White — Crunch › groupByKey()

