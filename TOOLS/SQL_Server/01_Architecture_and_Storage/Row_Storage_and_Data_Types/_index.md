---
id: "TOOLS/SQL_Server/01_Architecture_and_Storage/Row_Storage_and_Data_Types"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# Row Storage and Data Types

**L4 Advanced** · [↑ Architecture and Storage](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Every rowstore row is laid out as header + fixed-length columns + null bitmap + variable-length offsets and data, so the data types you pick decide row size, rows per page, and ultimately I/O and memory.

### 1. The big picture
Data types are not just validation rules; they are storage decisions. Each column's type fixes how many bytes it occupies (or may occupy) in every row, whether it lives in the fixed or variable part of the row, whether it can be stored off-row, and how efficiently it can be indexed, compared and joined. Because SQL Server reads whole 8 KB pages, a row that is 30% wider means ~30% more pages to scan, cache, back up and log.

The rule from the books is simple: **use the most restrictive data type that will never overflow.** Fixed length for values that are always the same length, variable length for values that vary, 8-bit strings when you can guarantee one code page, Unicode when you can't, and `MAX` types only when values may really exceed 8,000 bytes.

In a rowstore (heap or clustered B-tree), rows are written onto the page *one row after another, with every column of a row stored together*. Columnstore indexes do the opposite — that difference is why they compress and scan so well.

### 2. Core concepts and vocabulary
- **Record (row) header** — 4 bytes: status bits (record type, has null bitmap, has variable columns) and the offset to the end of the fixed-length part.
- **Fixed-length data** — columns like `INT`, `BIGINT`, `DATE`, `DATETIME2`, `DECIMAL`, `CHAR(n)`, `NCHAR(n)`, `BINARY(n)`: always take their full size, even when `NULL`.
- **Null bitmap** — 2-byte column count + 1 bit per column marking which are `NULL`.
- **Variable-length data** — `VARCHAR(n)`, `NVARCHAR(n)`, `VARBINARY(n)`, `sql_variant`, `MAX` types: take only the bytes used, plus 2 bytes each in the **column offset array**, plus a 2-byte count.
- **Code page / collation** — 8-bit (`CHAR`/`VARCHAR`) types store characters of one code page (or UTF-8 with a `_UTF8` collation, SQL Server 2019+). `NCHAR`/`NVARCHAR` store UTF-16 (2 bytes per character for most characters).
- **Row-overflow data** — when variable-length columns push a row past 8,060 bytes, some move to overflow pages with a 24-byte in-row pointer.
- **LOB data** — `VARCHAR(MAX)`, `NVARCHAR(MAX)`, `VARBINARY(MAX)`, `XML` values up to 2 GB; stored in-row if small enough, otherwise on LOB pages.
- **Deprecated types** — `TEXT`, `NTEXT`, `IMAGE`: replace with the `MAX` types.
- **Sparse column** — a column flagged `SPARSE` that takes no space when `NULL` but more space when not `NULL`.
- **User-defined / CLR types** — aliases like `Phone` or `Flag` for consistency, and built-in CLR types `HIERARCHYID`, `GEOMETRY`, `GEOGRAPHY`.

### 3. How it works, step by step
**Layout of one row (FixedVar format)**

| Part | Size |
|---|---|
| Status bits A and B | 2 bytes |
| Offset to end of fixed data | 2 bytes |
| Fixed-length columns | sum of their declared sizes |
| Column count | 2 bytes |
| Null bitmap | ceil(columns / 8) bytes |
| Variable column count | 2 bytes (if any variable columns are non-NULL) |
| Column offset array | 2 bytes × variable columns |
| Variable-length data | actual bytes |

Plus **2 bytes in the page's slot array** for each row.

**Example:** `Employees(EmployeeId INT, HireDate DATE, DepartmentCode NVARCHAR(2), LastName NVARCHAR(50))` with `'HR'` and `'Smith'`:
1. Header 4 bytes.
2. Fixed: `INT` 4 + `DATE` 3 = 7 bytes.
3. Column count 2 + null bitmap 1 = 3 bytes.
4. Variable count 2 + offsets 2 × 2 = 6 bytes.
5. Variable data: `'HR'` 4 bytes (UTF-16) + `'Smith'` 10 bytes = 14.
6. Total ≈ 34 bytes + 2-byte slot.

Change `DepartmentCode` to `NCHAR(2)` (it is *always* two characters): it moves to the fixed section (4 bytes) and no longer needs its 2-byte offset entry — **2 bytes saved per row**, which is the books' point: fixed-length is better when the length never varies.

**How `MAX` values are stored**
1. If the whole row (including the value) fits in 8,060 bytes, the `MAX` value is stored in-row by default.
2. Otherwise it goes to LOB pages and a pointer (16–24+ bytes) stays in-row.
3. `sp_tableoption 'large value types out of row', 1` forces off-row storage always — useful when large values are rarely read, so scans of other columns read fewer pages.

**String-function trap with `MAX`:** `REPLICATE('x', 10000)` returns `VARCHAR(8000)`, so it silently truncates to 8,000 characters. You must feed a `MAX` type in — `REPLICATE(CAST('x' AS VARCHAR(MAX)), 10000)` — to get 10,000. String functions return the type they receive.

### 4. Worked example
```sql
-- 1. Compare row sizes for different type choices
CREATE TABLE dbo.WideTypes (
    Id        BIGINT        NOT NULL,
    Code      NVARCHAR(10)  NOT NULL,   -- always 2 ASCII chars
    Price     DECIMAL(38,2) NOT NULL,
    CreatedAt DATETIME      NOT NULL,
    Notes     NCHAR(200)    NULL);      -- usually NULL
CREATE TABLE dbo.TightTypes (
    Id        INT           NOT NULL,
    Code      CHAR(2)       NOT NULL,
    Price     DECIMAL(9,2)  NOT NULL,
    CreatedAt DATETIME2(0)  NOT NULL,
    Notes     NVARCHAR(200) NULL);

INSERT dbo.WideTypes  SELECT TOP 100000 ROW_NUMBER() OVER (ORDER BY (SELECT 1)), 'HR', 19.99, SYSDATETIME(), NULL FROM sys.all_columns a CROSS JOIN sys.all_columns b;
INSERT dbo.TightTypes SELECT TOP 100000 ROW_NUMBER() OVER (ORDER BY (SELECT 1)), 'HR', 19.99, SYSDATETIME(), NULL FROM sys.all_columns a CROSS JOIN sys.all_columns b;

SELECT OBJECT_NAME(object_id) AS tbl, page_count, avg_record_size_in_bytes
FROM sys.dm_db_index_physical_stats(DB_ID(), NULL, NULL, NULL, 'DETAILED')
WHERE object_id IN (OBJECT_ID('dbo.WideTypes'), OBJECT_ID('dbo.TightTypes'));

-- 2. The MAX truncation trap
SELECT LEN(REPLICATE('x', 10000))                          AS truncated,   -- 8000
       LEN(REPLICATE(CAST('x' AS VARCHAR(MAX)), 10000))     AS full_length; -- 10000
```
Reading the result:
- `WideTypes` rows are around 450 bytes because `NCHAR(200)` takes 400 bytes even when `NULL`; `TightTypes` rows are about 30 bytes. That's roughly 17 vs 250 rows per page — a 15× difference in pages for every scan.
- `DECIMAL(38,2)` takes 17 bytes, `DECIMAL(9,2)` only 5; `DATETIME` 8 bytes vs `DATETIME2(0)` 6.

### 5. Under the hood
- **Fixed columns are stored even when NULL.** The null bitmap says the value is NULL, but the bytes are still reserved. Variable columns that are NULL (or trailing NULLs) take nothing.
- **Storage sizes to know:** `TINYINT` 1, `SMALLINT` 2, `INT` 4, `BIGINT` 8; `BIT` 1 byte per 8 bit columns; `DECIMAL` 5/9/13/17 bytes for precision 1–9/10–19/20–28/29–38; `DATE` 3; `TIME`/`DATETIME2` 3–5 / 6–8 by precision; `DATETIME` 8; `SMALLDATETIME` 4; `UNIQUEIDENTIFIER` 16; `MONEY` 8.
- **Unicode cost:** `NVARCHAR` uses 2 bytes per character (4 for supplementary characters). A UTF-8 collation on `VARCHAR` stores ASCII in 1 byte and other characters in 2–4.
- **Row-overflow:** only variable-length columns can move off-row; a row whose *fixed* part exceeds 8,060 bytes can't be created. Overflowed columns cost an extra page read each time they are needed.
- **Sparse columns:** a `NULL` sparse value takes 0 bytes; a non-`NULL` one costs extra (about 4 bytes of overhead plus the value). Worth it only when a column is `NULL` most of the time (often 60–90%+ depending on type). A `COLUMN_SET` exposes all sparse columns as one XML column for reading and writing.
- **Implicit conversions:** comparing an `NVARCHAR` parameter with a `VARCHAR` column forces a conversion on the column side, which can turn an index seek into a scan. Type choices must match application parameter types.
- **Columnstore contrast:** rowstore pages interleave all columns (Row1Col1…Row1Col6, Row2Col1…). A query summing one column still reads every column's bytes. Columnstore stores each column's values together, so the same query reads a fraction of the pages.

### 6. Performance and design implications
- Narrower rows → more rows per page → fewer logical reads, more of the table in cache, smaller indexes (clustering keys are copied into every nonclustered index).
- Right-size numerics and dates; don't default to `BIGINT`, `DECIMAL(38,x)`, `DATETIME`, `NVARCHAR(MAX)`.
- Unicode only when needed — but when data comes from the internet, partners or multiple collations, Unicode (or UTF-8 collations) avoids corruption, and correctness wins.
- Oversized declared lengths matter too: the optimizer estimates memory grants from *declared* sizes of variable columns (half the declared length), so `NVARCHAR(4000)` everywhere can over-grant memory for sorts and hashes.
- `MAX` columns can't be index key columns and block some online operations in older versions.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `sys.dm_db_index_physical_stats` → `avg/min/max_record_size_in_bytes` | Actual row sizes | Capacity and design review |
| `DATALENGTH(col)` | Bytes used by a value | Checking real data width |
| `sys.columns` (`max_length`, `system_type_id`) | Declared types and sizes | Auditing schemas |
| `ALTER TABLE … ALTER COLUMN` | Change a column's type | Right-sizing (may rewrite the table) |
| `sp_tableoption …, 'large value types out of row', 1` | Force MAX values off-row | Large, rarely read blobs |
| `col INT SPARSE NULL` / `COLUMN_SET FOR ALL_SPARSE_COLUMNS` | Sparse storage | Mostly-NULL attribute columns |
| `_UTF8` collations | UTF-8 in `VARCHAR` | Mostly-ASCII multilingual data |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `CHAR(n)`/`NCHAR(n)` — values always exactly n characters | Lengths vary → padding wastes space |
| `VARCHAR`/`NVARCHAR(n)` — lengths vary | Tiny always-fixed codes (offset array overhead) |
| Unicode — external/untrusted or multi-collation data | Guaranteed single code page data (2× storage) |
| `MAX` types — values may exceed 8,000 bytes | "Just in case" — can't index, off-row reads, bigger grants |
| `VARBINARY(MAX)` / FILESTREAM for files | Deprecated `IMAGE`, `TEXT`, `NTEXT` |
| `SPARSE` — mostly NULL columns | Columns frequently non-NULL (more space, slower reads) |

### 9. Common mistakes
- **Using `NVARCHAR(MAX)` for every string** → no indexing, inflated memory grants, off-row reads → size to expected data.
- **Variable-length for fixed codes** → 2 extra bytes per row per column → use `CHAR(2)`/`NCHAR(2)`.
- **`BIGINT`/`DECIMAL(38,…)` by default** → 2–3× wider rows and indexes → pick the smallest safe type.
- **Forgetting `REPLICATE`/string functions return non-`MAX` types** → silent truncation at 8,000 characters → cast the input to `MAX`.
- **Mismatched parameter types (`NVARCHAR` vs `VARCHAR` column)** → implicit conversion, index scans → match types in application code.
- **Still using `TEXT`/`IMAGE`** → deprecated, functions like `LEFT` fail → migrate to `VARCHAR(MAX)`/`VARBINARY(MAX)`.

### 10. 🎯 Interview kit
**30-second answer:** A rowstore row is a 4-byte header, the fixed-length columns at full size, a null bitmap, then an offset array and the variable-length values; each row also uses a 2-byte slot. So type choice decides row width, rows per page and I/O. Use the most restrictive type that won't overflow: fixed-length for constant-length values, variable for varying, Unicode only when needed, and `MAX` only for truly large values, which may be stored off-row.

**Structure a longer answer:** row layout → fixed vs variable cost → Unicode vs 8-bit → off-row (row-overflow, LOB) → side effects (indexing, memory grants, implicit conversion) → scenario: shrinking a fact table by right-sizing types.

**Likely follow-ups:**
- *Does a NULL `CHAR(100)` take space?* → yes, 100 bytes; a NULL `VARCHAR(100)` doesn't.
- *What happens when a row exceeds 8,060 bytes?* → variable columns move to row-overflow pages.
- *When are sparse columns worth it?* → when the column is NULL for the large majority of rows.
- *Why does `NVARCHAR` vs `VARCHAR` matter for queries?* → implicit conversion can prevent seeks.

**Signals interviewers look for:** you calculate row sizes, link types to pages and memory, and balance storage efficiency against correctness (Unicode, overflow risk).

### 11. Check yourself
- [ ] Calculate the size of a row with `INT`, `DATE`, `CHAR(2)` and a 20-character `VARCHAR(50)`.
- [ ] Explain why changing a two-character code from `NVARCHAR(2)` to `NCHAR(2)` saves 2 bytes.
- [ ] Show the `REPLICATE` truncation and fix it.
- [ ] Decide between `VARCHAR`, `NVARCHAR` and a UTF-8 collation for a customer-name column fed by a website.
- [ ] Describe when a sparse column saves space and when it wastes it.
<!-- explainer:end -->

## References

- *Expert Performance Indexing in SQL Server* — Jason Strate, Grant Fritchey — Large Object Page
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Expanding on Data Type Concepts › Large-Value Binary Data Types
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Expanding on Data Type Concepts › Sparse Columns
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Storage and Performance Considerations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Common Pitfalls and Mistakes
- *Analytics Optimization with Columnstore Indexes in Microsoft SQL Server* — Edward Pollack — Columnstore Index arChIteCture R HTE TURE › Physical Data on Pages
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Introduction
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Data types › 3.2 #7 Always using variable-length strings
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Structure

