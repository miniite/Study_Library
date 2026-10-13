---
id: "TOOLS/SQL_Server/06_T-SQL_Programming/JSON_and_XML"
level: 4
type: "curriculum"
curriculum: "sql_server"
---

# JSON and XML

**L4 Advanced** · [↑ T-SQL Programming](<../_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** SQL Server can store, query, shred and generate semi-structured data: JSON (stored as text, or as the native `json` type in SQL Server 2025/Azure SQL) with `JSON_VALUE`, `JSON_QUERY`, `OPENJSON` and `FOR JSON`; and XML with the `xml` type, XQuery methods, `FOR XML` and XML indexes. Use them at the edges (APIs, flexible attributes, exchange formats) while keeping core data relational.

### 1. The big picture
Applications talk JSON. Older integrations and config files talk XML. Sooner or later a database receives a document or must produce one. Ignoring these features leads to two bad outcomes: parsing documents row by row in the application, or storing blobs nobody can query. SQL Server lets you:
- **Store** documents (`nvarchar(max)` with `ISJSON` checks, the `json` type, or the `xml` type).
- **Query** values inside documents (`JSON_VALUE`, `.value()`).
- **Shred** documents into rows and columns (`OPENJSON`, `.nodes()`).
- **Generate** documents from queries (`FOR JSON`, `FOR XML`).
- **Index** them (computed columns for JSON, XML indexes, JSON indexes in 2025).

The design rule: if you filter, join or aggregate on an attribute regularly, it belongs in a real column. Documents are for variable, sparse or pass-through data.

### 2. Core concepts and vocabulary
- **JSON**: JavaScript Object Notation: objects `{"key": value}`, arrays `[…]`, strings, numbers, `true/false`, `null`.
- **JSON path**: an expression that addresses a value: `$.customer.name`, `$.items[0].qty`. `$` is the document root. Prefix `strict` errors if the path is missing; default `lax` returns NULL.
- **`ISJSON(x)`**: 1 if valid JSON. Use in a `CHECK` constraint.
- **`JSON_VALUE(doc, path)`**: extracts a *scalar* (string up to 4000 chars by default; `RETURNING` type in 2025).
- **`JSON_QUERY(doc, path)`**: extracts an *object or array* as JSON text.
- **`JSON_MODIFY(doc, path, value)`**: returns the document with a value changed, added or removed.
- **`OPENJSON(doc [, path]) [WITH (…)]`**: table-valued function that shreds JSON into rows. Without `WITH`: key/value/type rows. With `WITH`: typed columns mapped by paths.
- **`FOR JSON PATH | AUTO`**: turns a query result into JSON text. `PATH` uses column aliases like `'customer.name'` to build nesting.
- **Native `json` type** (SQL Server 2025, Azure SQL): stores JSON in an optimized binary format, validated on write, with `JSON_OBJECTAGG`/`JSON_ARRAYAGG` and JSON indexes. Older books state SQL Server has no JSON type; that was true up to SQL Server 2022.
- **`xml` data type**: stores XML in an internal binary form; optionally **typed** by an XML schema collection.
- **XQuery methods** on `xml`: `.value()` (scalar), `.query()` (XML fragment), `.exist()` (bit), `.nodes()` (shred into rows), `.modify()` (XML DML).
- **`FOR XML RAW | AUTO | PATH | EXPLICIT`**: produce XML from queries. `FOR XML PATH('')` was the classic string-concatenation trick before `STRING_AGG`.
- **Primary / secondary XML indexes**: shred XML into an internal node table to speed XQuery.

### 3. How it works, step by step
**Storing and validating JSON (pre-2025):**
1. Column `nvarchar(max)` with `CHECK (ISJSON(doc) = 1)`.
2. Every read parses the text. There is no binary storage, so repeated `JSON_VALUE` calls parse the document each time.
3. To filter fast, add a **computed column** on `JSON_VALUE(doc, '$.path')` and index it. The optimizer matches the same expression in your `WHERE` to the index.

**Shredding with OPENJSON:**
1. Input: JSON text (variable or column).
2. Optional second argument moves to a sub-path (e.g. `'$.items'`).
3. `WITH (col type '$.path', …)` defines output columns; `AS JSON` keeps a nested object/array as text for further `OPENJSON` via `CROSS APPLY`.
4. Each array element becomes a row.
5. Requires database compatibility level 130+.

**Generating with FOR JSON PATH:**
1. Each result row becomes an object; the result is an array.
2. Dotted aliases (`[customer.name]`) create nested objects.
3. A subquery with `FOR JSON PATH` in the SELECT list creates a nested array.
4. Options: `ROOT('name')`, `INCLUDE_NULL_VALUES`, `WITHOUT_ARRAY_WRAPPER`.

### 4. Worked example
```sql
CREATE TABLE dbo.WebOrders
(
    orderid  int IDENTITY PRIMARY KEY,
    received datetime2 NOT NULL DEFAULT SYSDATETIME(),
    doc      nvarchar(max) NOT NULL CONSTRAINT CK_WebOrders_doc CHECK (ISJSON(doc) = 1)
);

INSERT dbo.WebOrders (doc) VALUES
(N'{"customer":{"id":71,"name":"Save-a-lot"},"channel":"web",
    "items":[{"sku":"A-1","qty":2,"price":9.5},{"sku":"B-7","qty":1,"price":20}]}');

-- Scalars and fragments
SELECT  JSON_VALUE(doc, '$.customer.name') AS customer,
        JSON_VALUE(doc, '$.items[0].sku')  AS first_sku,
        JSON_QUERY(doc, '$.items')         AS items_json
FROM dbo.WebOrders;

-- Shred the items array into rows
SELECT  w.orderid, i.sku, i.qty, i.price, i.qty * i.price AS line_total
FROM dbo.WebOrders AS w
CROSS APPLY OPENJSON(w.doc, '$.items')
     WITH (sku varchar(20) '$.sku', qty int '$.qty', price decimal(10,2) '$.price') AS i;

-- Index a frequently filtered attribute
ALTER TABLE dbo.WebOrders
  ADD custid AS CAST(JSON_VALUE(doc, '$.customer.id') AS int);
CREATE INDEX IX_WebOrders_custid ON dbo.WebOrders (custid);

SELECT orderid FROM dbo.WebOrders WHERE custid = 71;    -- index seek

-- Modify in place
UPDATE dbo.WebOrders
SET doc = JSON_MODIFY(doc, '$.channel', 'mobile')
WHERE orderid = 1;

-- Generate nested JSON for an API
SELECT  c.custid      AS [id],
        c.companyname AS [name],
        (SELECT o.orderid AS [id], o.orderdate AS [date]
         FROM Sales.Orders AS o
         WHERE o.custid = c.custid
         FOR JSON PATH) AS [orders]
FROM Sales.Customers AS c
WHERE c.custid = 71
FOR JSON PATH, WITHOUT_ARRAY_WRAPPER;
```

**XML equivalents:**
```sql
DECLARE @x xml = N'<order id="1"><item sku="A-1" qty="2"/><item sku="B-7" qty="1"/></order>';

SELECT  @x.value('(/order/@id)[1]', 'int')           AS orderid,
        n.c.value('@sku', 'varchar(20)')             AS sku,
        n.c.value('@qty', 'int')                     AS qty
FROM @x.nodes('/order/item') AS n(c);                -- one row per <item>

SELECT custid AS [@id], companyname AS [name]
FROM Sales.Customers
WHERE custid <= 2
FOR XML PATH('customer'), ROOT('customers');
```
`.value()` needs a singleton path, hence `[1]`.

### 5. Under the hood
- **Text JSON** is parsed on every function call. Several `JSON_VALUE` calls on the same large document repeat the parse; one `OPENJSON … WITH` parses once and returns all columns.
- **Native `json`** (2025) is stored in a binary format, so path access doesn't reparse text; documents up to 2 GB; `CREATE JSON INDEX` indexes paths. Check compatibility and client driver support before switching.
- **Computed column indexing**: the computed column need not be persisted; the index stores the value. The query's expression must match the column's expression (same path, same `CAST`), or reference the column directly.
- **`JSON_VALUE` returns `nvarchar(4000)`**. Comparing to an `int` triggers implicit conversion; CAST explicitly.
- **XML type** is stored as a binary representation up to 2 GB. Without XML indexes, each XQuery shreds the blob at runtime (plans show *Table Valued Function [XML Reader]* operators with poor estimates). A **primary XML index** persists the shredded node table (often 2–3× the XML size); secondary indexes (PATH, VALUE, PROPERTY) speed specific query shapes. **Selective XML indexes** index only chosen paths.
- **FOR JSON / FOR XML** build output in a streaming way; large results come back as a single string split into 2,033-character chunks by some clients.
- **STRING_AGG** (SQL 2017) replaces the `FOR XML PATH('')` concatenation trick, without XML entity encoding problems (`&` → `&amp;`).

### 6. Performance and design implications
- Promote hot attributes to real (or computed + indexed) columns. Filtering `WHERE JSON_VALUE(doc,'$.x') = …` without an index scans and parses every row.
- Shred once with `OPENJSON … WITH`; avoid many `JSON_VALUE` calls on large documents.
- Use `OPENJSON` to pass arrays of values to procedures (an alternative to table-valued parameters and string splitting).
- Large documents in `nvarchar(max)` go off-row (LOB pages), increasing I/O; keep documents small or split them.
- XML indexes cost storage and slow writes; add them only when XQuery is frequent.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ISJSON(x)` in CHECK | Validates JSON on write | Text JSON columns |
| `JSON_VALUE` / `JSON_QUERY` | Scalar / object-array extraction | Reading attributes |
| `JSON_MODIFY` | Change a value in a document | Small edits |
| `OPENJSON … WITH` | Shred JSON to rows | Imports, parameters |
| `FOR JSON PATH` | Generate JSON | APIs |
| `json` type, `CREATE JSON INDEX` (2025) | Native binary JSON and path indexes | New designs on 2025 / Azure SQL |
| `xml.value/.nodes/.exist/.query/.modify` | XQuery on XML | XML data |
| `FOR XML PATH` | Generate XML | Legacy integrations |
| `CREATE PRIMARY XML INDEX` | Persist shredded XML | Frequent XQuery |
| `STRING_AGG(col, ',')` | Concatenate strings | Replace FOR XML PATH trick |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| JSON column for variable/sparse attributes, API payloads | Attributes you join/filter/aggregate on constantly → columns |
| Native `json` (2025) for new JSON-heavy designs | Older clients/tools that expect text, or older compat levels |
| `OPENJSON` to pass lists to procedures | Huge lists → bulk load or TVPs |
| `xml` type for schema-validated documents | New designs where JSON fits better |
| XML indexes | Write-heavy tables with rare XML queries |

### 9. Common mistakes
- Storing everything as JSON "for flexibility" → no constraints, slow queries → model core entities relationally.
- `JSON_VALUE` on an object/array → returns NULL → use `JSON_QUERY`.
- `lax` path with a typo → silent NULLs → use `strict` while developing.
- No `ISJSON` check → invalid documents break later queries → CHECK constraint.
- `.value()` without `[1]` → error "requires a singleton".
- Using `FOR XML PATH('')` for CSV lists → `&` becomes `&amp;` → `STRING_AGG`, or `.value('.', 'nvarchar(max)')`.
- Filtering on `JSON_VALUE` with type mismatch → implicit conversion, scans → CAST in the computed column.

### 10. 🎯 Interview kit
**30-second answer:** Before 2025, SQL Server stores JSON as nvarchar with ISJSON checks; you read it with JSON_VALUE and JSON_QUERY, shred it with OPENJSON, generate it with FOR JSON, and index hot paths through computed columns. SQL Server 2025 and Azure SQL add a native binary json type and JSON indexes. XML has its own type, XQuery methods like value and nodes, FOR XML, and XML indexes. Keep frequently queried attributes as real columns.

**Structure a longer answer:** when documents make sense → storage options → querying/shredding → indexing strategy → generating output for APIs → a real case (e.g. passing an array of IDs via OPENJSON).

**Likely follow-ups:**
- *How do you index JSON?* → computed column on JSON_VALUE + index; JSON index in 2025.
- *JSON_VALUE vs. JSON_QUERY?* → scalar vs. object/array.
- *lax vs. strict?* → NULL vs. error for missing paths.
- *How to shred an array?* → `CROSS APPLY OPENJSON(doc, '$.items') WITH (…)`.

**Signals interviewers look for:** you don't put core relational data in blobs; you know the indexing pattern; you're aware of version differences (2016 JSON, 2025 native type).

### 11. Check yourself
- [ ] Store an order document with a CHECK constraint and shred its items into rows.
- [ ] Add an indexed computed column on a JSON path and confirm an index seek.
- [ ] Generate nested JSON (customer with an orders array) with FOR JSON PATH.
- [ ] Shred an XML document with `.nodes()` and `.value()`.
- [ ] Explain when you'd keep an attribute in JSON vs. promote it to a column.
<!-- explainer:end -->

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Working with JSON: OPENJSON, JSONVALUE, FOR JSON
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Working with XML and JSON › JSON Data
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Parsing JSON into Rows and Columns: OPENJSON
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Working with XML and JSON › 15. Working with XML and JSON
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Best Practices for JSON in SQL Server
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Extracting Values from JSON: JSONVALUE and JSONQUERY
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › JSONQUERY
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 6: CTEs, MERGE, Pivoting, JSON, and XML › Understanding JSON in SQL Server
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Working with XML and JSON › Inserting JSON Data
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Working with XML and JSON › Shredding JSON Data

