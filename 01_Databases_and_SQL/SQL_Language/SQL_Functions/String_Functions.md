---
id: "01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Functions"
tools: ["MySQL", "SQL_Server"]
---

# String Functions

**L4 Advanced** · [↑ SQL Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** String functions find, cut, clean, join, pad, replace and convert text inside SQL — useful for cleaning data on the way in, but each one changes types, NULL behavior and index usability in ways that differ by engine.

### 1. The big picture
Text data is the messiest data: padded fixed-length columns, stray spaces, mixed case, phone numbers in six formats, codes packed into one column. String functions are the toolbox for turning that into something joinable and groupable. Beginning T-SQL notes you often have a choice of *where* to manipulate a string — in the database, or in the client — and that "it might make sense to utilize the client to do the work if the manipulation is complex". Fuzzy Data Matching with SQL makes the opposite point: for matching and cleansing, you live in these functions.

The functions fall into six groups:

| Group | Typical functions |
|---|---|
| Measure | `LEN`/`LENGTH`, `DATALENGTH`/`OCTET_LENGTH`, `CHAR_LENGTH` |
| Locate | `CHARINDEX`, `PATINDEX`, `INSTR`, `POSITION`, `LIKE` |
| Extract | `SUBSTRING`/`SUBSTR`, `LEFT`, `RIGHT` |
| Clean/transform | `TRIM`, `LTRIM`, `RTRIM`, `UPPER`, `LOWER`, `REPLACE`, `REVERSE`, `TRANSLATE`, `INITCAP`, regex functions |
| Build | `CONCAT`, `CONCAT_WS`, `||`, `+`, `LPAD`/`RPAD`, `STRING_AGG`/`GROUP_CONCAT`/`LISTAGG`, `FORMAT`/`TO_CHAR` |
| Convert | `CAST`, `CONVERT`, `TRY_CAST`, `ASCII`, `CHAR` |

### 2. Core concepts and vocabulary
- **CHAR(n) vs VARCHAR(n)** — fixed-length (space-padded) vs variable-length. `CHAR` columns need trimming when you compare or concatenate.
- **Unicode types** — `NVARCHAR`/`NCHAR` (SQL Server) hold UTF-16; `VARCHAR` with a UTF-8 collation or PostgreSQL `text` hold UTF-8. Lengths in characters and in bytes differ.
- **1-based positions** — SQL string positions start at 1; `CHARINDEX` returns 0 (not -1, not NULL) when not found (Fuzzy Data Matching: "1-relative strings… programmers used to 0-relative beware!").
- **Collation** — rules for comparison/sorting (case/accent sensitivity). `CHARINDEX('B','abcd')` is 2 under a case-insensitive collation and 0 under a case-sensitive one (Beginning T-SQL).
- **Pattern vs regex** — `LIKE`/`PATINDEX` use `%` and `_` wildcards; regex functions (`REGEXP_REPLACE`, `REGEXP_LIKE`, `~`) are far more expressive and mostly engine-specific.
- **Implicit conversion** — the engine silently converts one operand to the other's type when types differ, following precedence rules.
- **String aggregation** — collapsing many rows into one delimited string per group.

### 3. How it works, step by step
**Core functions with 1-based semantics (T-SQL flavour; equivalents in §7):**

```sql
SELECT LEN('  abc  ')                    AS len_ignoring_trailing_spaces, -- 5 (T-SQL ignores TRAILING spaces)
       DATALENGTH('abc')                AS bytes,                        -- 3 (varchar)
       CHARINDEX('c', 'abcabc')         AS first_c,                      -- 3
       CHARINDEX('c', 'abcabc', 4)      AS next_c,                       -- 6 (start at position 4)
       SUBSTRING('abcdef', 2, 3)        AS sub,                          -- 'bcd'  (start, LENGTH)
       LEFT('abcdef', 2)                AS l,                            -- 'ab'
       RIGHT('abcdef', 2)               AS r,                            -- 'ef'
       REPLACE('a-b-c', '-', '')        AS rep,                          -- 'abc'
       REVERSE('abc')                   AS rev,                          -- 'cba'
       UPPER('Ana')                     AS up,
       TRIM('  x  ')                    AS t;                            -- 'x' (SQL Server 2017+)
```

**Concatenation and NULL** — the classic trap:

| Expression | NULL behaviour |
|---|---|
| `a + b` (SQL Server) | NULL if either is NULL (with `CONCAT_NULL_YIELDS_NULL ON`, the default) |
| `a \|\| b` (PostgreSQL, Oracle, SQLite, ANSI) | NULL if either NULL in PostgreSQL/ANSI; **Oracle treats `''` as NULL and `\|\|` skips NULLs** |
| `CONCAT(a,b)` (SQL Server, PostgreSQL) | NULLs treated as empty string |
| `CONCAT(a,b)` (MySQL) | **NULL if any argument is NULL** |
| `CONCAT_WS(sep, a, b, …)` | Skips NULL arguments; no stray separators |

(Some answer-bank text claims "CONCAT returns NULL if any argument is NULL" for all engines. That is true for MySQL only.)

### 4. Worked example
**Clean and standardize a messy customer table:**
```sql
SELECT customer_id,
       UPPER(LEFT(TRIM(first_name),1)) + LOWER(SUBSTRING(TRIM(first_name),2,100)) AS first_name_clean,
       CONCAT_WS(' ', TRIM(first_name), TRIM(last_name))                          AS full_name,
       -- keep digits only (SQL Server 2017+: TRANSLATE trick; PostgreSQL: regexp_replace(phone,'\D','','g'))
       REPLACE(REPLACE(REPLACE(REPLACE(phone,'(',''),')',''),'-',''),' ','')      AS phone_digits,
       CHARINDEX('@', email)                                                      AS at_pos,
       SUBSTRING(email, CHARINDEX('@', email) + 1, LEN(email))                    AS email_domain,
       RIGHT('00000' + CAST(zip AS varchar(5)), 5)                                AS zip5   -- left-pad to 5
FROM   staging.customers;
```

Walking through it: `TRIM` first (otherwise leading blanks defeat `LEFT`); `CONCAT_WS` avoids a trailing space when `last_name` is NULL; nested `REPLACE` strips punctuation (works, but see performance); `CHARINDEX('@', …)` returns 0 for invalid emails, so wrap the substring in `CASE WHEN CHARINDEX('@',email) > 0 …` in real code; zero-padding with `RIGHT('00000'+…, 5)` is the T-SQL idiom where `LPAD` doesn't exist (`LPAD(zip::text, 5, '0')` in PostgreSQL/MySQL/Oracle).

**Aggregate strings per group:**
```sql
-- SQL Server 2017+ / PostgreSQL / Snowflake
SELECT dept, STRING_AGG(name, ', ') WITHIN GROUP (ORDER BY name) AS members FROM emp GROUP BY dept;
-- MySQL
SELECT dept, GROUP_CONCAT(name ORDER BY name SEPARATOR ', ') FROM emp GROUP BY dept;
-- Oracle
SELECT dept, LISTAGG(name, ', ') WITHIN GROUP (ORDER BY name) FROM emp GROUP BY dept;
```
Ordering is only guaranteed if you specify it; limits differ — MySQL `group_concat_max_len` silently truncates at 1024 bytes by default; Oracle `LISTAGG` errors on overflow past 4000 bytes unless you add `ON OVERFLOW TRUNCATE` (12.2+).

**Data-type conversion pitfalls (Ultimate T-SQL):**
```sql
SELECT 10 + 5.75;                 -- int → decimal, 15.75
SELECT CASE WHEN '123' = 123 THEN 'Equal' END;   -- 'Equal': varchar converted to int
SELECT CASE WHEN 'ABC' = 123 THEN 'Equal' END;   -- ERROR: cannot convert 'ABC' to int
```
Implicit conversion converts the *lower-precedence* type to the higher; strings lose to numbers, so a non-numeric string raises a runtime error.

### 5. Under the hood
- **Padding semantics.** `CHAR(10)` stores `'abc       '`. SQL Server ignores trailing spaces in equality comparisons (`'abc' = 'abc   '` is true under ANSI padding) and `LEN` excludes them, while `DATALENGTH` counts them. PostgreSQL treats `char(n)` similarly but `text`/`varchar` strictly.
- **Chars vs bytes.** `LEN('héllo')` is characters, `DATALENGTH` is bytes (5 vs 6 in UTF-8; 10 in `nvarchar`). Length-limited columns overflow by bytes in some engines (MySQL `utf8mb4` uses up to 4 bytes per char).
- **Collation-dependent comparison.** `UPPER(a) = UPPER(b)` is often unnecessary; a case-insensitive collation already does it, and it avoids killing the index.
- **Regex engines.** PostgreSQL uses POSIX regex; MySQL 8 uses ICU; Oracle uses its own; patterns aren't portable (`\d` vs `[[:digit:]]`).
- **`ISNUMERIC` false positives.** SQL Server's `ISNUMERIC('1e4')`, `ISNUMERIC('$')`, `ISNUMERIC('.')`, `ISNUMERIC(',')` return 1 because *some* numeric type accepts them. Use `TRY_CAST(x AS int)` / `TRY_CONVERT` (returns NULL on failure) or a pattern check `x NOT LIKE '%[^0-9]%'`.
- **Large objects.** `REPLACE`/`SUBSTRING` on `varchar(max)` can't be pushed through indexes and may spill to LOB storage; CLR/regex functions are slower per row than native ones.

### 6. Performance and design implications
- **Don't wrap indexed columns.** `WHERE LEFT(name,3)='Smi'` or `WHERE UPPER(email)=…` becomes a scan; `LIKE 'Smi%'` (leading prefix only) is sargable, `LIKE '%mith'` isn't. The Ultimate T-SQL implicit-conversion section adds that comparing a `VARCHAR` column with an `INT` literal converts the *column* and makes the index useless: match types (`WHERE Phone = '1234567890'`).
- **Cleanse at write time, not query time.** Standardizing phone numbers with nested `REPLACE`/`REGEXP_REPLACE` in every query costs CPU per row per query and is hard to keep consistent; do it once in the ETL or a persisted computed column (and index that).
- **Avoid `FORMAT()` on big sets** in SQL Server (CLR-backed; 10–50× slower than `CONVERT`); format in the presentation tier.
- **String aggregation** over very large groups builds big strings in memory; consider returning rows and aggregating in the app.
- **Sargable alternatives**: store a normalized/lowercased column; use `COLLATE` once; full-text index for word search; trigram index (`pg_trgm`) for `LIKE '%x%'`.
- **CAST vs CONVERT.** `CAST` is ANSI; `CONVERT(type, value, style)` is T-SQL-specific and offers date/number styles. Use `CAST` for plain type change, `CONVERT` when you need a format style, and `TRY_` variants when input is untrusted. Using `CAST` where a format change is needed is a common mistake (it can't format).

### 7. Commands and settings you should know
| Task | SQL Server | PostgreSQL | MySQL | Oracle |
|---|---|---|---|---|
| Length (chars) | `LEN` | `char_length`, `length` | `CHAR_LENGTH` (`LENGTH` = bytes) | `LENGTH` |
| Find position | `CHARINDEX(sub, s)` | `strpos`, `position(sub in s)` | `INSTR(s, sub)`, `LOCATE` | `INSTR(s, sub)` |
| Substring | `SUBSTRING(s, start, len)` | `substr`, `substring(s from 2 for 3)` | `SUBSTRING`, `SUBSTR` | `SUBSTR` |
| Pad | `RIGHT('0'+x, n)` | `lpad`, `rpad` | `LPAD`, `RPAD` | `LPAD`, `RPAD` |
| Trim | `TRIM`, `LTRIM`, `RTRIM` | `trim`, `btrim` | `TRIM` | `TRIM` |
| Capitalize words | custom | `initcap` | custom | `INITCAP` |
| Replace | `REPLACE`, `TRANSLATE` | `replace`, `translate`, `regexp_replace` | `REPLACE`, `REGEXP_REPLACE` | `REPLACE`, `REGEXP_REPLACE` |
| Concat | `CONCAT`, `CONCAT_WS`, `+` | `concat`, `concat_ws`, `\|\|` | `CONCAT`, `CONCAT_WS` | `\|\|`, `CONCAT` (2 args) |
| Aggregate | `STRING_AGG` | `string_agg` | `GROUP_CONCAT` | `LISTAGG` |
| Safe convert | `TRY_CAST`, `TRY_CONVERT` | `CAST` + validation / `::` | `CAST` (warns) | `CAST ... DEFAULT ON CONVERSION ERROR` |
| Char codes | `ASCII`, `CHAR`, `UNICODE`, `NCHAR` | `ascii`, `chr` | `ASCII`, `CHAR` | `ASCII`, `CHR` |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| Clean in ETL/staging once and store the result | Re-cleaning in every report |
| `CONCAT_WS` for optional parts | `+`/`\|\|` when NULLs are possible and you didn't plan for them |
| `TRY_CAST` for dirty input | You need to detect and log bad values (it hides them) |
| Computed/persisted normalized column + index | Pattern searches in the middle of text (use full-text or trigram) |
| Regex for validation/extraction | Hot paths and portable SQL |
| Aggregating strings in SQL for small groups | Result size is unbounded |

### 9. Common mistakes
- `WHERE col = NULL`-style thinking applied to strings: concatenating a NULL and getting NULL (`first + ' ' + last`) → blank names in reports → `CONCAT_WS` or `COALESCE`.
- Forgetting 1-based indexing → off-by-one in `SUBSTRING`/`LEFT` → test boundaries.
- `CHARINDEX` returning 0 and being used as a length → `SUBSTRING(s, 1, -1)` error → guard with `CASE`.
- Comparing `VARCHAR` with `NVARCHAR` or numbers → implicit conversion warnings, scans → align types.
- Using `ISNUMERIC` as a validator → '1e4', '$' pass → `TRY_CAST`.
- Applying `UPPER`/`LOWER` to both sides of a join → no index use → fix collation or normalize on write.
- Assuming `LEN` counts trailing spaces → pad-sensitive logic breaks → know `DATALENGTH`.
- `GROUP_CONCAT` truncation unnoticed → silently incomplete lists → raise `group_concat_max_len`.

### 10. 🎯 Interview kit
**30-second answer:** "String functions measure, locate, extract, clean, build and convert text: `LEN`, `CHARINDEX`/`INSTR`, `SUBSTRING`, `TRIM`, `REPLACE`, `CONCAT`/`||`, `STRING_AGG`, `CAST`. Names vary by engine; what matters is 1-based positions, NULL propagation in concatenation, trailing-space semantics, and that wrapping an indexed column in a function makes the predicate non-sargable."

**Structure a longer answer:** groups of functions → one cleaning example → NULL and type pitfalls → performance (sargability, write-time cleansing).

**Likely follow-ups:**
- *`CONCAT` vs `||`/`+`?* → NULL handling differs per engine; `CONCAT_WS` skips NULLs.
- *`CAST` vs `CONVERT`?* → ANSI vs T-SQL with styles; `TRY_` forms.
- *`LTRIM` vs `RTRIM`?* → left/right side whitespace; `TRIM` both.
- *Pad to fixed length?* → `LPAD`/`RPAD` or `RIGHT('000'+x,n)`.
- *Standardize phone numbers?* → at write time, regex/`TRANSLATE`, store normalized column.
- *`STRING_AGG`/`GROUP_CONCAT`/`LISTAGG` differences?* → ordering syntax, size limits, overflow behavior.
- *Why is `ISNUMERIC` unreliable?* → accepts currency symbols/exponent; use `TRY_CAST`.

**Signals interviewers look for:** you mention NULL behavior, collation/trailing spaces, and index impact without being prompted.

### 11. Check yourself
- [ ] Extract the domain from an email safely when '@' may be missing.
- [ ] Predict `CONCAT`, `+`, and `||` results with a NULL in SQL Server, MySQL, PostgreSQL, Oracle.
- [ ] Explain why `WHERE Phone = 1234567890` is slow when `Phone` is `VARCHAR`.
- [ ] Pad a number to 6 digits in two engines.
- [ ] Replace `ISNUMERIC` with a safer check and explain why.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../../../TOOLS/MySQL/_index.md>)
- [SQL Server](<../../../TOOLS/SQL_Server/_index.md>)

## Practice questions

- [Basic] [Write a query using UPPER and LOWER to standardize the case of a name column.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41153_Write_a_query_using_UPPER_and_LOWER_to_standardize_the_case_.md>)
- [Basic] [What is the difference between LTRIM and RTRIM?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41208_What_is_the_difference_between_LTRIM_and_RTRIM.md>)
- [Basic] [What does the REVERSE function do and when might it be useful?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41313_What_does_the_REVERSE_function_do_and_when_might_it_be_usefu.md>)
- [Intermediate] [What is the difference between CONCAT and CONCAT_WS?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41304_What_is_the_difference_between_CONCAT_and_CONCAT_WS.md>)
- [Intermediate] [You must standardize inconsistent phone numbers using nested REPLACE/REGEXP_REPLACE. What are the performance and correctness risks of doing this at query time versus at write time?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41194_You_must_standardize_inconsistent_phone_numbers_using_nested.md>)
- [Deep] [How would you extract the first name from a full name column that has a space separator?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41350_How_would_you_extract_the_first_name_from_a_full_name_column.md>)
- [Deep] [You need to aggregate strings per group. Compare STRING_AGG, GROUP_CONCAT, and LISTAGG in terms of ordering guarantees, size limits, and overflow handling.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41200_You_need_to_aggregate_strings_per_group_Compare_STRING_AGG_G.md>)
- [Deep] [How would you detect that ISNUMERIC is giving false positives (e.g., for '1e4' or '$') and what safer alternatives exist for validating numeric input?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41234_How_would_you_detect_that_ISNUMERIC_is_giving_false_positive.md>)

## Related concepts

- [String Functions](<../SQL_Fundamentals/String_Functions.md>) *(SQL Fundamentals)*
- [CTEs in SQL Server](<../../../TOOLS/SQL_Server/SQL_Server_Platform/CTEs_in_SQL_Server.md>) *(SQL Server Platform)*
- [Charset and Collation Migration](<../../../TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration.md>) *(MySQL Platform)*

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — 4. Using Built-in Functions and Expressions › Expressions Using Operators
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Using Mathematical Operators › Using String Functions
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Using Mathematical Operators › LEN and DATALENGTH
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Using Mathematical Operators › CHARINDEX
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › Conversion Functions
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › String Functions
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › LEN
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Function Junction › LEFT, RIGHT, and SUBSTRING
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Implicit Conversion Examples
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Performance Issues with Implicit Conversion
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 8: Query Optimization and Performance Tuning › Implicit Conversions
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 13: Security, Permissions, and Auditing › Masking Functions in SQL Server
- *T-SQL Fundamentals (4th ed.)* — Itzik Ben-Gan — Single-table queries (cont.)
- *Hadoop: The Definitive Guide* — Tom White — Hive › Conversions
- *Learning Spark (2nd ed.)* — Jules S. Damji, Brooke Wenig, Tathagata Das, Denny Lee — Spark SQL and DataFrames Interacting with External Data Sources › Built-in Functions for Complex Data Types
- *SQL Server Tutorial* (YouTube series) — Part 22: Built in string functions in sql server 2008 ([watch](https://www.youtube.com/watch?v=qJFr-R76r9A))
- *SQL Server Tutorial* (YouTube series) — Part 24: Replicate, Space, Patindex, Replace and Stuff string functions in sql server 2008 ([watch](https://www.youtube.com/watch?v=ALnM6d7OQUs))
- *SQL Server Tutorial* (YouTube series) — Part 123: TRY PARSE function in SQL Server 2012 ([watch](https://www.youtube.com/watch?v=Z2eZwddqMEU))

