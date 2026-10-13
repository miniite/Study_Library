---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/String_Functions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: ["SQL_Server", "MySQL"]
---

# String Functions

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** String functions let you measure, slice, join, clean and split text in SQL; the recurring traps are `NULL` propagation, trailing-space and collation rules, differing function names across engines, and wrapping an indexed column in a function so the index can't be used.

### 1. The big picture
Real data is text-heavy: names, emails, codes, URLs, delimited lists, free-form comments. Cleaning and reshaping it is the daily work of ETL and analytics. SQL offers a small standard core (`SUBSTRING`, `TRIM`, `UPPER`, `LOWER`, `REPLACE`, `CONCAT`, `LIKE`, `CAST`) and a long tail of vendor-specific functions (`LEN` vs `LENGTH` vs `CHAR_LENGTH`, `CHARINDEX` vs `POSITION` vs `STRPOS`, `STRING_SPLIT` vs `string_to_array`/`unnest`, `+` vs `||`).

Three principles carry you across engines:
1. **Know the NULL rule** of each concatenation form.
2. **Know whether the function is sargable** (can an index still be used?).
3. **Know the unit**: characters vs bytes, and the collation that governs comparison and case.

### 2. Core concepts and vocabulary
- **Character vs byte length** — `LEN`/`CHAR_LENGTH` count characters; `DATALENGTH`/`OCTET_LENGTH` count bytes (UTF-8 and `NVARCHAR` change the answer).
- **CHAR(n) vs VARCHAR(n)** — fixed-width, space-padded vs variable-width.
- **Collation** — rules for sorting and comparing text: case sensitivity, accent sensitivity.
- **Concatenation** — joining strings (`CONCAT`, `||`, `+`).
- **Delimited string** — several values packed in one column (`'a;b;c'`).
- **Sargable** — a predicate form that lets an index seek.
- **Wildcards** — `%` any sequence, `_` one character in `LIKE`; escape with `ESCAPE`.
- **Implicit conversion** — the engine silently casting between text and numbers/dates.

### 3. How it works, step by step
**Measure.** `LEN(s)` (SQL Server; ignores trailing spaces), `LENGTH(s)` (PostgreSQL/MySQL/Oracle), `CHAR_LENGTH(s)` (standard). `DATALENGTH` / `OCTET_LENGTH` for bytes.

**Slice.** `SUBSTRING(s, start, length)` is 1-based. `LEFT(s, n)`, `RIGHT(s, n)`. Find positions with `CHARINDEX(sub, s)` (SQL Server), `POSITION(sub IN s)` (standard), `STRPOS` (PostgreSQL), `INSTR` (Oracle/MySQL).

**Join.**
- `CONCAT(a, b, …)` treats `NULL` as an empty string.
- `a || b` (standard; PostgreSQL, Oracle*, SQLite) yields `NULL` if either operand is `NULL` (*Oracle treats empty string as NULL, so its `||` skips it).
- `a + b` (SQL Server) yields `NULL` when either is `NULL` under `CONCAT_NULL_YIELDS_NULL ON` (the default).
- `CONCAT_WS(sep, …)` joins with a separator and skips `NULL`s.

**Clean.** `TRIM`/`LTRIM`/`RTRIM`, `UPPER`/`LOWER`, `REPLACE(s, old, new)` (replaces all occurrences, case-sensitive per collation), `TRANSLATE` for character maps, regex functions (`REGEXP_REPLACE` in PostgreSQL/Oracle/MySQL; none native in SQL Server before 2025).

**Split / aggregate.**
- SQL Server 2016+: `STRING_SPLIT(s, ';')` returns a table with column `value`; the `enable_ordinal` argument (2022+) adds the position. `STRING_AGG(col, ',')` for the reverse.
- PostgreSQL: `unnest(string_to_array(s, ';'))`, `split_part(s, ';', n)`, `string_agg`.
- BigQuery/Spark: `SPLIT` + `UNNEST` / `EXPLODE` (use `EXPLODE_OUTER` / `LEFT JOIN UNNEST` to keep rows with empty arrays).

**Convert.** `CAST(s AS INT)` fails on bad input; `TRY_CAST`/`TRY_CONVERT` (SQL Server), `try_cast` (Snowflake/Spark/Trino) or a regex guard avoid errors.

### 4. Worked example
```sql
-- Sample data
-- id | full_name            | email              | tags
-- 1  | '  Ada  Lovelace '   | 'ADA@Example.com'  | 'sql;etl'
-- 2  | 'Grace Hopper'       | NULL               | ''
-- 3  | 'Linus'              | 'linus@x.org'      | NULL

-- Clean and split a name (SQL Server)
SELECT id,
       LTRIM(RTRIM(full_name))                                         AS clean_name,
       LEFT(LTRIM(RTRIM(full_name)),
            CHARINDEX(' ', LTRIM(RTRIM(full_name)) + ' ') - 1)         AS first_name,
       LOWER(email)                                                    AS email_norm
FROM   people;
```

Walk-through: `+ ' '` guarantees `CHARINDEX` finds a space even for single-word names like `'Linus'`, so `LEFT(..., 0)` doesn't happen. `LOWER(email)` returns `NULL` for row 2: scalar string functions propagate `NULL`.

NULL-safe concatenation:

```sql
SELECT id,
       first_name + ' ' + last_name                 AS naive,      -- NULL if either is NULL
       CONCAT(first_name, ' ', last_name)           AS concat_fn,  -- 'Ada ' for missing last name
       CONCAT_WS(' ', first_name, last_name)        AS tidy        -- skips NULLs, no stray space
FROM   people2;
```

Split a delimited list into rows without dropping empty/NULL cases:

```sql
-- SQL Server
SELECT p.id, s.value AS tag
FROM   people p
OUTER  APPLY STRING_SPLIT(p.tags, ';') s;           -- keeps id 3 (NULL) with NULL tag; id 2 gives ''

-- PostgreSQL
SELECT p.id, t.tag
FROM   people p
LEFT   JOIN LATERAL unnest(string_to_array(p.tags, ';')) AS t(tag) ON true;
-- An inner CROSS JOIN would silently drop rows whose list is NULL/empty.
```

Safe numeric parse:

```sql
SELECT raw_value,
       TRY_CAST(raw_value AS DECIMAL(12,2)) AS parsed        -- NULL on failure (SQL Server)
FROM   staging;
-- Portable guard: CASE WHEN raw_value ~ '^[0-9]+(\.[0-9]+)?$' THEN raw_value::numeric END   -- PostgreSQL
```

Escape wildcards: `WHERE code LIKE '10\%%' ESCAPE '\'` finds codes starting with the literal `10%`.

### 5. Under the hood
- **Sargability.** `WHERE LOWER(email) = 'ada@example.com'` cannot seek an index on `email` (engine must compute `LOWER` per row). Fixes: use a case-insensitive collation on the column, store a normalized column, or create a functional/computed index (`CREATE INDEX ON t (LOWER(email))` in PostgreSQL; indexed persisted computed column in SQL Server). `LIKE 'abc%'` can seek; `LIKE '%abc'` cannot (needs full scan or trigram/full-text index).
- **Trailing spaces.** `'abc' = 'abc  '` is true under ANSI padding rules for `CHAR` and in SQL Server for `VARCHAR` comparison (trailing blanks ignored), which also explains why `LEN` ignores them. PostgreSQL `varchar` comparison treats them as significant.
- **Unicode.** `VARCHAR` (code page / UTF-8) vs `NVARCHAR` (UTF-16). Literals need the `N'...'` prefix in SQL Server or non-ASCII characters can be replaced by `?`. Some characters (emoji, combining marks) are multiple code units, so `LEN` and `SUBSTRING` can split a character.
- **Memory.** `VARCHAR(MAX)`/`TEXT` values are stored off-row beyond ~8 KB; functions over them are slower and can't be index keys.
- **Implicit conversion.** Comparing a `VARCHAR` column to an `NVARCHAR` parameter in SQL Server converts the column (higher type precedence) and destroys the seek: match parameter and column types.

### 6. Performance and design implications
- Clean once at load time into typed, normalized columns; don't repeat `LOWER(TRIM(x))` in every query.
- Never store multiple values in one column if you query them; model a child table instead. Splitting is for ingestion.
- Prefer set-based functions over row-by-row procedural parsing; for heavy parsing use regex or the warehouse's native array/JSON functions.
- Use `LIKE` prefix searches or full-text/trigram indexes for search; don't rely on `%term%` scans on large tables.
- Test with `NULL`, empty string, leading/trailing spaces, mixed case and non-ASCII values.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `LEN` / `LENGTH` / `CHAR_LENGTH` | String length | Validation, padding |
| `SUBSTRING`, `LEFT`, `RIGHT` | Slice | Parsing fixed layouts |
| `CHARINDEX` / `POSITION` / `STRPOS` | Find substring | Split by first delimiter |
| `CONCAT`, `CONCAT_WS`, `\|\|` | Join strings | Building labels |
| `TRIM` / `LTRIM` / `RTRIM` | Remove spaces (or chars) | Clean user input |
| `REPLACE`, `TRANSLATE`, `REGEXP_REPLACE` | Substitute | Normalization |
| `STRING_SPLIT`, `string_to_array`+`unnest`, `SPLIT`+`EXPLODE` | Delimited → rows | Ingestion |
| `STRING_AGG` | Rows → delimited | Reporting |
| `TRY_CAST` | Convert or NULL | Dirty input |
| `LIKE … ESCAPE` | Literal wildcards | Searching `%`/`_` |
| `CONCAT_NULL_YIELDS_NULL` (SQL Server) | NULL rule for `+` | Legacy behaviour |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `CONCAT`/`CONCAT_WS` for display strings | You *want* NULL to signal a missing component (use `+`/`||` deliberately) |
| Computed/functional index for case-insensitive lookups | Writes are heavy and the column is wide |
| Case-insensitive collation | Mixed-collation joins (collation conflict errors, slow plans) |
| Regex for validation | Simple prefix tests (`LIKE`) are cheaper and sargable |
| Split at ingest into a child table | The list is truly opaque and never filtered |

### 9. Common mistakes
- `a + b` with a NULL → whole result NULL → `CONCAT`, `COALESCE`, or `CONCAT_WS`.
- `CROSS APPLY STRING_SPLIT` on NULL lists → rows disappear → `OUTER APPLY`.
- `WHERE UPPER(col) = …` on a big table → scan → collation or functional index.
- `REPLACE` for case-insensitive replace assuming case-sensitivity → depends on collation → check and test.
- `LEN` to measure storage → ignores trailing spaces and counts characters → `DATALENGTH`.
- `CAST` of dirty strings → job fails at row 40 million → `TRY_CAST` and quarantine failures.
- `LIKE '%_%'` meaning "contains underscore" → `_` is a wildcard → `ESCAPE`.

### 10. 🎯 Interview kit
**30-second answer:** Core string work is slice (`SUBSTRING`, `LEFT`), find (`CHARINDEX`/`POSITION`), clean (`TRIM`, `LOWER`, `REPLACE`), join (`CONCAT`, `||`) and split (`STRING_SPLIT`/`unnest`). I handle `NULL` explicitly (`CONCAT` treats it as empty; `+`/`||` propagate it), avoid wrapping indexed columns in functions, and use `TRY_CAST` for dirty input.

**Structure a longer answer:** list the operations needed → vendor names → NULL and collation rules → sargability → set-based vs row-based → example (split full name / explode tags without dropping rows).

**Likely follow-ups:**
- *Difference between `CONCAT` and `||`?* → NULL handling and engine support.
- *How do you split a delimited column into rows?* → `STRING_SPLIT`/`unnest`; preserve empties with outer apply/left join.
- *Why is `LOWER(col)=…` slow?* → non-sargable; fix with functional index/collation.
- *How do you validate numeric strings?* → `TRY_CAST` or regex guard.

**Signals interviewers look for:** NULL discipline, awareness of engine differences, performance consciousness, and robustness to dirty data.

### 11. Check yourself
- [ ] Predict the result of `'a' + NULL`, `CONCAT('a', NULL)` and `'a' || NULL`.
- [ ] Split `'a;b;;c'` into rows and explain what happens to the empty element.
- [ ] Make a case-insensitive email lookup use an index.
- [ ] Explain why `LEN('abc   ')` is 3 in SQL Server.
- [ ] Safely convert a dirty text column to `DECIMAL` and list the failures.
<!-- explainer:end -->

## Implementations (tools)

- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)
- [MySQL](<../../../TOOLS/MySQL/_index.md>)

## Practice questions

- [Basic] [How would you find the length of a string stored in a column, and which function does your RDBMS use?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39776_How_would_you_find_the_length_of_a_string_stored_in_a_column.md>)
- [Basic] [How would you extract the first three characters of a string column using SUBSTRING?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39779_How_would_you_extract_the_first_three_characters_of_a_string.md>)
- [Basic] [How would you replace all occurrences of one substring with another inside a column using REPLACE?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39781_How_would_you_replace_all_occurrences_of_one_substring_with_.md>)
- [Basic] [What does the CONCAT function do and how does it differ from using the || or + concatenation operator?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39790_What_does_the_CONCAT_function_do_and_how_does_it_differ_from.md>)
- [Basic] [How do you concatenate two columns into a single string in a query, and how do NULLs affect it?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39978_How_do_you_concatenate_two_columns_into_a_single_string_in_a.md>)
- [Basic] [How do you write a SQL query to split a full_name column into first_name and last_name during transformation?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39998_How_do_you_write_a_SQL_query_to_split_a_full_name_column_int.md>)
- [Intermediate] [How do you convert a string like '2026-01-15' into a proper date type, and what can go wrong?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40011_How_do_you_convert_a_string_like_2026-01-15_into_a_proper_da.md>)
- [Intermediate] [Why can casting a string column to a number in SQL fail silently or error, and how do you safely handle bad values for a feature?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40592_Why_can_casting_a_string_column_to_a_number_in_SQL_fail_sile.md>)

## Related concepts

- [String Functions](<../SQL_Functions/String_Functions.md>) *(SQL Functions)*
- [CTEs in SQL Server](<../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>) *(SQL Server Platform)*
- [Cleaning Text and Numeric Fields](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>) *(Data Preprocessing)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 7: Dynamic SQL, Triggers, and Table-Valued Functions (TVFs) › Escaping Strings with REPLACE()
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Writing Simple SELECT Queries › Mixing Literals and Column Names
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 4. Using Built-in Functions and Expressions › CONCAT
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 4. Using Built-in Functions and Expressions › Concatenating Other Data Types to Strings
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)

