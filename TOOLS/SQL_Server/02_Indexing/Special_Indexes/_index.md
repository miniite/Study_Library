---
id: "TOOLS/SQL_Server/02_Indexing/Special_Indexes"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# XML Spatial and Full-Text Indexes

**L4 Advanced** · [↑ Indexing](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** For data that B-trees can't search well (words inside text, XML documents, shapes on a map, JSON, vectors), SQL Server offers specialized indexes: **full-text**, **XML**, **spatial**, and in SQL Server 2025 **JSON** and **vector**. Each pre-processes the data into a searchable structure, and each needs its own query syntax to be used.

### 1. The big picture
A B-tree works when you search by a value, or by the *beginning* of a value, in a column. Many real searches don't look like that:
- "Find documents containing *invoice* near *overdue*", where words are buried inside text. `LIKE '%invoice%'` scans every row, because the leading wildcard defeats the B-tree.
- "Find orders whose XML has `/Order/Item[@sku="A1"]`", where the value is inside a document.
- "Find stores within 5 km of this point", which needs a 2-D proximity search that sorted values can't express.
- "Find rows whose JSON `$.customer.id` = 7", or "find the 10 most similar embeddings".

Each special index works the same basic way: **decompose** the complex value into many small, indexable pieces (words, XML nodes, grid cells, JSON paths, graph neighbours), store those pieces in an internal structure (often hidden B-tree tables), and give queries a matching **predicate syntax** (`CONTAINS`, XQuery methods, `STIntersects`, `JSON_VALUE`/`JSON_CONTAINS`, `VECTOR_SEARCH`). Use the wrong syntax, and the special index is ignored.

### 2. Core concepts and vocabulary
- **Full-text index**: an inverted index of words (*tokens*) → rows, stored in a **full-text catalog**. Built and maintained by the full-text engine (FDHOST processes), not by the normal index code.
- **Word breaker / stemmer**: language-specific components that split text into words and find inflectional forms (run/ran/running).
- **Stoplist**: words ignored as noise ("the", "and").
- **KEY INDEX**: the unique, single-column, non-nullable index a full-text index uses to identify rows. A narrow `int` key is best.
- **Change tracking (full-text)**: `AUTO` (background updates), `MANUAL`, or `OFF` (full repopulation only).
- **Primary XML index**: a persisted, shredded representation of every node of each XML value (a hidden clustered B-tree, the "node table").
- **Secondary XML indexes**: `PATH`, `VALUE` and `PROPERTY` B-trees built on the primary XML index, for path-based, value-based and property-retrieval queries.
- **Selective XML index**: (2012 SP1+) indexes only chosen paths, so it is much smaller than a primary XML index.
- **Spatial index**: maps each shape onto a multi-level **grid** (tessellation) and stores the cells it touches in a B-tree. Used by `geometry` and `geography` methods.
- **Tessellation levels / CELLS_PER_OBJECT**: four grid levels, each `LOW/MEDIUM/HIGH` density, plus a limit on cells recorded per shape.
- **Bounding box**: the required extent for `geometry` grids (`geography` covers the globe).
- **JSON index**: (SQL Server 2025, preview) indexes paths inside a native `json` column.
- **Vector index**: (SQL Server 2025, preview) an approximate-nearest-neighbour index (DiskANN) on `vector` columns.

### 3. How it works, step by step
**Full-text:**
1. Create a catalog, then `CREATE FULLTEXT INDEX ON t (col LANGUAGE 1033) KEY INDEX pk ON catalog WITH CHANGE_TRACKING AUTO`.
2. Population: the engine reads rows, word-breaks each document, drops stopwords, and writes (word, key, position) entries into internal index fragments. Fragments are periodically merged ("master merge").
3. Queries use **`CONTAINS`** (precise: words, phrases, prefix `"data*"`, `NEAR`, inflectional forms) or **`FREETEXT`** (meaning-based: stems and thesaurus). `CONTAINSTABLE` and `FREETEXTTABLE` also return a **RANK**.
4. The full-text engine returns matching keys, which join back to the base table through the KEY INDEX.
5. Updates are asynchronous: results lag behind changes until the background population catches up.

**XML:**
1. The table needs a clustered primary key.
2. `CREATE PRIMARY XML INDEX` shreds every XML value into rows (node id, tag, value, path…). This often makes it 2–5× the size of the XML itself.
3. Secondary indexes target query shapes: **PATH** for `exist('/a/b[@c="x"]')` with known paths, **VALUE** for searching values when the path is wildcarded (`//b[.="x"]`), and **PROPERTY** for retrieving several values from one row's XML via `value()`.
4. Queries must use XQuery methods (`exist()`, `value()`, `query()`, `nodes()`) for the optimizer to use these indexes.

**Spatial:**
1. Choose a grid: four levels of increasingly fine cells (e.g. 4×4, 8×8 or 16×16 per level).
2. Each shape is *tessellated*: the cells it covers or touches are recorded, up to `CELLS_PER_OBJECT`. The covering and deepest-cell rules decide when to stop subdividing.
3. A spatial query (`@area.STIntersects(geo) = 1`, `geo.STDistance(@p) < 5000`) first uses the index as a **primary filter**: rows whose cells overlap the query's cells. Candidates may be false positives.
4. The **secondary filter** runs the exact geometric test on the candidates.
5. The quality of the primary filter (how few false positives) depends on the grid density and the size of the shapes.

**JSON / vector (2025):** `CREATE JSON INDEX ix ON t(jsoncol) FOR ('$.customer.id')` speeds up `JSON_VALUE`, `JSON_PATH_EXISTS` and `JSON_CONTAINS` predicates. `CREATE VECTOR INDEX … WITH (METRIC = 'cosine', TYPE = 'diskann')` serves `VECTOR_SEARCH` approximate top-k queries. Both are new, so check version notes before relying on them. The older pattern of a persisted computed column `AS JSON_VALUE(doc,'$.id')` plus a normal B-tree still works everywhere.

### 4. Worked example
```sql
-- Full-text
CREATE FULLTEXT CATALOG ftDocs;
CREATE FULLTEXT INDEX ON dbo.Documents (Body LANGUAGE 1033)
    KEY INDEX PK_Documents ON ftDocs WITH CHANGE_TRACKING AUTO;

SELECT d.DocID, k.RANK
FROM CONTAINSTABLE(dbo.Documents, Body, 'NEAR((invoice, overdue), 5)') AS k
JOIN dbo.Documents AS d ON d.DocID = k.[KEY]
ORDER BY k.RANK DESC;
-- vs: WHERE Body LIKE '%invoice%'  → full scan of every row's text

-- XML
CREATE PRIMARY XML INDEX PXML_Orders ON dbo.Orders(OrderXml);
CREATE XML INDEX SXML_Orders_Path ON dbo.Orders(OrderXml)
    USING XML INDEX PXML_Orders FOR PATH;
SELECT OrderID FROM dbo.Orders
WHERE OrderXml.exist('/Order/Item[@sku="A1"]') = 1;

-- Spatial
CREATE SPATIAL INDEX SIX_Stores_Location ON dbo.Stores(Location)
    USING GEOGRAPHY_AUTO_GRID WITH (CELLS_PER_OBJECT = 12);
DECLARE @p geography = geography::Point(47.61, -122.33, 4326);
SELECT StoreID FROM dbo.Stores WHERE Location.STDistance(@p) <= 5000;

-- JSON, portable pattern (any version)
ALTER TABLE dbo.Events ADD CustomerId AS CAST(JSON_VALUE(Payload,'$.customer.id') AS int) PERSISTED;
CREATE INDEX IX_Events_CustomerId ON dbo.Events(CustomerId);
```
What to check: plans for full-text show a *Remote Scan / Table Valued Function [FulltextMatch]* operator. XML plans show seeks on the hidden node tables (`xml_index_nodes_…`). Spatial plans show a *Clustered Index Seek (Spatial)* followed by a Filter for the exact test. If you see a plain scan of the base table, your predicate shape didn't match the index.

### 5. Under the hood
- **Full-text internals.** Index fragments are like immutable mini-indexes. Each population or tracked change adds a fragment, and a **master merge** combines them (`ALTER FULLTEXT CATALOG … REORGANIZE` forces one). Many fragments mean slower queries. Inspect them in `sys.fulltext_index_fragments`, and see population status in `sys.dm_fts_index_population`. Metadata lives in `sys.fulltext_indexes`, `sys.fulltext_index_columns` and `sys.fulltext_catalogs`.
- **Prefix searches** (`"data*"`) work in full-text. Suffix/infix searches (`*base`) do not. For these, consider n-gram tables or a different tool.
- **XML index size and maintenance.** The primary XML index stores every node, so updates to large XML values rewrite many rows. Selective XML indexes, or promoting hot values into computed columns with a function, are often cheaper. Metadata is in `sys.xml_indexes` (and `sys.selective_xml_index_paths`).
- **Spatial tuning.** Large shapes in a fine grid hit `CELLS_PER_OBJECT` and stop at coarse levels, giving more false positives. Small points in a coarse grid also filter poorly. `sp_help_spatial_geography_index` / `sp_help_spatial_geometry_index` report the primary filter efficiency for a sample query. `*_AUTO_GRID` (2012+) uses 8 levels and is a sound default. Metadata: `sys.spatial_indexes`, `sys.spatial_index_tessellations`.
- **All of them appear in `sys.indexes`** with distinctive `type_desc` values (`XML`, `SPATIAL`; full-text appears separately in `sys.fulltext_indexes`). Their internal tables show up in `sys.internal_tables`.

### 6. Performance and design implications
- Use a full-text index instead of `LIKE '%word%'` for word searches over large text. Accept the asynchronous lag, or use `MANUAL` tracking with scheduled populations for heavy-write tables.
- XML indexes are heavy. Index XML only when queries filter *inside* it often. Otherwise shred to relational columns at load time.
- For spatial, test grid choices with the help procedures and real queries. Points (e.g. store locations) usually work well with auto grids.
- Store JSON as `json` (2025) or `nvarchar(max)`, and index the hot paths with computed columns or a JSON index. Don't scan and parse documents for every query.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `CREATE FULLTEXT CATALOG / INDEX … KEY INDEX …` | Word index over text columns | Searching words in text |
| `CONTAINS`, `FREETEXT`, `CONTAINSTABLE`, `FREETEXTTABLE` | Full-text predicates and ranking | Text search queries |
| `ALTER FULLTEXT INDEX … START FULL POPULATION` | Rebuild full-text content | After bulk changes / manual tracking |
| `CREATE PRIMARY XML INDEX` / `CREATE XML INDEX … FOR PATH/VALUE/PROPERTY` | XML node indexes | Frequent XQuery filters |
| `CREATE SELECTIVE XML INDEX … FOR (path = '…')` | Index chosen paths only | Large XML, few hot paths |
| `CREATE SPATIAL INDEX … USING GEOGRAPHY_AUTO_GRID` | Grid-based spatial index | Distance/intersection queries |
| `sp_help_spatial_geography_index` | Primary filter efficiency | Spatial tuning |
| Persisted computed column + B-tree on `JSON_VALUE` | Index one JSON path | JSON lookups (any version) |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Full-text: word/phrase/prefix search on large text | Need exact substring/suffix matches, or instant consistency |
| Primary + secondary XML: varied XQuery filters | Large XML that changes often (huge index maintenance) |
| Selective XML / computed columns: a few hot paths | Many unpredictable paths |
| Spatial index: proximity/intersection at scale | Tiny tables (the exact test on all rows is cheap) |
| Computed column on JSON path | Paths change often (schema drift) |

### 9. Common mistakes
- `LIKE '%term%'` on millions of documents → full scans → full-text `CONTAINS`.
- Expecting full-text results immediately after an insert → missing rows → understand async population, monitor `sys.dm_fts_index_population`.
- Writing `OrderXml.value(...) = 'x'` in a WHERE clause and expecting index use → often a full shred per row → use `exist()` with the predicate inside the path.
- Wrapping a geography column in other functions, or using `STDistance` without a constant bound → no index use → write `col.STDistance(@p) <= @d`.
- Using a wide GUID as the full-text KEY INDEX → larger index and slower joins → use a narrow integer key.

### 10. 🎯 Interview kit
**30-second answer:** "B-trees can't search inside text, documents or geometry, so SQL Server has special indexes. Full-text is an inverted word index queried with CONTAINS/FREETEXT. XML indexes shred XML into node tables for XQuery. Spatial indexes tessellate shapes onto a multi-level grid and then run an exact filter. SQL Server 2025 adds JSON and vector indexes. Each needs its matching predicate syntax."
**Structure a longer answer:** why B-trees fail (leading wildcards, nested values, 2-D) → decompose-and-index idea → one example per type → maintenance costs → how to verify use in plans.
**Likely follow-ups:** *Why doesn't `LIKE '%x%'` use an index?* → no usable leading prefix in the sort order. *Primary vs secondary XML index?* → the shredded node table vs targeted B-trees on it. *Primary vs secondary filter in spatial?* → grid overlap candidates vs the exact geometric test.
**Signals interviewers look for:** knowing when *not* to use these (shredding, computed columns), awareness of async full-text, and correct predicate shapes.

### 11. Check yourself
- [ ] Explain an inverted index with a three-document example.
- [ ] Rewrite a `LIKE '%term%'` query using `CONTAINS` and compare the plans.
- [ ] Choose PATH vs VALUE vs PROPERTY secondary XML indexes for three queries.
- [ ] Describe spatial tessellation and the primary/secondary filter steps.
- [ ] Index a JSON property in a version-independent way.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Full-Text Search Index Catalog Views and Properties
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Dropping an Index › sys.xml:indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Dropping an Index › sys.spatialindexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — sys.fulltextindexcolumns
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Index Physical Stats Summary › Categories
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Creating a Full-Text Catalog
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — The Views
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Syntax
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Spatial Indexes
- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Page Compression › LIKE Comparison

