---
id: "TOOLS/MySQL/MySQL_Platform/Charset_and_Collation_Migration"
level: 4
type: "concept"
topic: "TOOLS/MySQL/MySQL_Platform"
tools: ["MySQL"]
---

# Charset and Collation Migration

**L4 Advanced** · [↑ MySQL Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** A character set decides which bytes represent which characters, a collation decides how those characters compare and sort, and moving a live table from `latin1`/`utf8` to `utf8mb4` is a table rebuild that changes index sizes, uniqueness, and join behaviour — so you plan it, not just run it.

### 1. The big picture
Every string column in MySQL carries two settings: a **character set** (the encoding — how many bytes each character takes) and a **collation** (the comparison rules — case, accents, sort order). They look like cosmetic settings until something breaks: an emoji is silently dropped or rejected, `'a' = 'A'` is true when you expected false, a join stops using its index, or a `UNIQUE` index refuses a perfectly valid row.

The book material agrees on the core ideas. A collation defines the character set, case sensitivity, accent sensitivity and sort order, and comparing columns with *different* collations forces implicit conversions that slow queries and can change results (T-SQL Fundamentals/Ultimate T-SQL — *Collation and Sorting Rules*). Fuzzy-matching practice adds that collations drive not only `ORDER BY` but also `<`, `>` and `<>`, and that "bad characters" arriving from CSV or external feeds are best handled by deciding on a Unicode type up front (*Fuzzy Data Matching with SQL* — Ch. 8). The vendor-neutral lesson transfers directly to MySQL; the MySQL-specific details (the `utf8` trap, the 3,072-byte index limit, `ALTER ... CONVERT`) are what this lesson adds.

This topic sits under the MySQL platform lessons next to online schema changes (the migration is a rebuild) and constraint definitions (unique keys are collation-dependent).

### 2. Core concepts and vocabulary
- **Character set (charset)** — a repertoire of characters plus their byte encoding, e.g. `latin1`, `utf8mb3`, `utf8mb4`.
- **Collation** — a set of comparison/sort rules for one charset, e.g. `utf8mb4_0900_ai_ci`. Naming: `_ci` case-insensitive, `_cs` case-sensitive, `_ai` accent-insensitive, `_as` accent-sensitive, `_bin` compare raw bytes.
- **`utf8` / `utf8mb3`** — MySQL's *legacy* UTF-8 that stores at most 3 bytes per character. It cannot hold 4-byte characters (emoji, some CJK extensions). `utf8` is an alias for `utf8mb3` and is deprecated.
- **`utf8mb4`** — real UTF-8, 1–4 bytes per character; the default charset in MySQL 8.0 (default collation `utf8mb4_0900_ai_ci`). MySQL 5.7 defaulted to `latin1`.
- **PAD SPACE vs NO PAD** — whether trailing spaces are ignored in `=` comparison. Older collations (`utf8mb4_general_ci`, `utf8mb4_unicode_ci`) are PAD SPACE; the `0900` family is NO PAD.
- **`CHAR_LENGTH()` vs `LENGTH()`** — characters vs bytes. `CHAR_LENGTH('é')` is 1; `LENGTH('é')` is 2 in utf8mb4; an emoji is 1 char / 4 bytes.
- **Connection charset** — `character_set_client`, `character_set_connection`, `character_set_results`; set together by `SET NAMES utf8mb4`.
- **Index key length limit** — InnoDB allows 3,072 bytes per index key with `DYNAMIC`/`COMPRESSED` row format (767 bytes for old `COMPACT`/`REDUNDANT`).
- **Mojibake** — garbled text from bytes interpreted in the wrong charset (`Ã©` instead of `é`).

### 3. How it works, step by step — a safe migration to utf8mb4
1. **Inventory.** Find every column, table and schema default that is not `utf8mb4`:
   `SELECT table_schema, table_name, column_name, character_set_name, collation_name FROM information_schema.columns WHERE character_set_name IS NOT NULL AND character_set_name <> 'utf8mb4';`
2. **Check what is really stored.** A `latin1` column may actually contain UTF-8 bytes written by an app that never set the connection charset. Converting such a column "properly" double-encodes it. Test on a copy with `SELECT HEX(col)` and compare.
3. **Check index and row-size impact.** `VARCHAR(255)` in `utf8mb3` needs up to 765 bytes; in `utf8mb4` up to 1,020. Composite indexes sum their parts against the 3,072-byte cap. The row-size cap (65,535 bytes across non-BLOB columns) is also based on the *maximum* bytes per character.
4. **Check uniqueness under the new collation.** `utf8mb4_0900_ai_ci` treats `'resume'` and `'résumé'` as equal. A unique index on such data can fail with `Duplicate entry` during conversion. Find them first with `GROUP BY col COLLATE utf8mb4_0900_ai_ci HAVING COUNT(*) > 1`.
5. **Align the join partners.** Foreign-key columns must have matching charset/collation; joined columns should too, or MySQL converts one side.
6. **Rebuild the table.** `ALTER TABLE t CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;` rewrites every row and every index. This is not an instant/in-place operation, so on big busy tables use an online schema-change tool (gh-ost, pt-online-schema-change) or a shadow-table copy.
7. **Change the defaults** — schema, table, and server (`character-set-server`, `collation-server`) — so *new* objects match.
8. **Fix the clients.** Drivers and ORMs must send `utf8mb4` (`charset=utf8mb4` in the connection string / `SET NAMES utf8mb4`). Otherwise you convert the table and keep writing mojibake.
9. **Verify.** Re-run the inventory, run `EXPLAIN` on the hot joins, check `SHOW WARNINGS`, and round-trip an emoji and an accented value through the app.

### 4. Worked example
```sql
-- Why emoji fail in utf8 (utf8mb3)
CREATE TABLE msg_old (id INT PRIMARY KEY, body VARCHAR(100)) CHARACTER SET utf8mb3;
SET NAMES utf8mb4;
INSERT INTO msg_old VALUES (1, 'hi 😀');   -- strict mode: ERROR 1366 Incorrect string value
                                           -- non-strict: truncated at the emoji, with a warning

-- Length functions on multibyte text
SELECT CHAR_LENGTH('café 😀') AS chars,    -- 6
       LENGTH('café 😀')      AS bytes;    -- 10 (c,a,f =3; é=2; space=1; emoji=4)

-- Collation controls comparison
SELECT 'Hello' = 'HELLO' COLLATE utf8mb4_0900_ai_ci AS ci,   -- 1
       'Hello' = 'HELLO' COLLATE utf8mb4_0900_as_cs AS cs,   -- 0
       'e'     = 'é'     COLLATE utf8mb4_0900_ai_ci AS ai,   -- 1
       'e'     = 'é'     COLLATE utf8mb4_0900_as_cs AS ax;   -- 0

-- Migration
ALTER TABLE msg_old CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
```
Walk-through: the first insert fails (or truncates) because the 4-byte emoji cannot be encoded in `utf8mb3`. After the `CONVERT`, the same insert succeeds and `CHAR_LENGTH`/`LENGTH` show the difference between characters and bytes. The collation row shows that "case-insensitive" and "accent-insensitive" are separate switches.

To find a charset-mismatch join that silently lost its index:
```sql
EXPLAIN SELECT * FROM orders o JOIN legacy_customers c ON o.customer_code = c.customer_code;
SHOW WARNINGS;   -- look for convert(... using utf8mb4) wrapped around the column
```
When one side is wrapped in a conversion, the index on that side cannot be used for the lookup, and the plan shows a scan (`type: ALL`/`index`) instead of `ref`/`eq_ref`.

### 5. Under the hood
- **Why 3,072 and 767.** InnoDB stores long keys in B+tree pages (16 KB). The old 767-byte limit was tied to `COMPACT`/`REDUNDANT` row formats and 5.6 behaviour (`innodb_large_prefix` allowed up to 3,072 with `DYNAMIC`). On 8.0 with default `DYNAMIC` rows you get 3,072 bytes per key. That's why `VARCHAR(255)` in `utf8mb4` (1,020 bytes) fits a single-column index, while a three-column composite of `VARCHAR(255)` does not (3,060 bytes plus other parts may exceed it). A **prefix index** (`INDEX(url(191))`) is the escape hatch — at the cost of not being usable as a covering index and not enforcing full-length uniqueness.
- **Where the conversion happens.** When the two operands of a comparison have different collations, MySQL applies *coercibility* rules to pick one; column values have higher precedence than literals, and an error ("Illegal mix of collations") is raised when two columns of the same coercibility disagree and neither charset is a superset. When charsets differ but one is a superset (`utf8mb3` ⊂ `utf8mb4`), MySQL converts the narrower side to the wider one — which, when the narrower side is the indexed column, defeats the index.
- **`0900` vs older collations.** `utf8mb4_0900_*` uses Unicode 9.0 UCA weights, is NO PAD, and is generally faster than `utf8mb4_unicode_ci` for comparisons. NO PAD means `'a' = 'a '` is false; moving from `unicode_ci` to `0900_ai_ci` can therefore change which rows match and may un-collide or collide unique keys differently.
- **Binary collations.** `utf8mb4_bin` compares bytes of the code points: fastest, exact, and case/accent-sensitive; use it for tokens, hashes, and identifiers where fuzzy equality is wrong.
- **Storage.** `CHAR(n)` in a variable-width charset reserves up to n × maxlen bytes in the row-size calculation, but InnoDB stores it as variable length in practice; `VARCHAR` stores the actual bytes plus a 1–2 byte length prefix.

### 6. Performance and design implications
- **Index fan-out** drops as keys get longer: fewer entries per page, deeper trees, larger buffer-pool footprint. Wider charset = bigger indexes, even for pure-ASCII data (the *stored* size of ASCII in utf8mb4 is still 1 byte per char, but the *maximum* is used for limit checks and for in-memory sort buffers and temp tables, where `VARCHAR` may be padded to max length).
- **Temp tables and sorts** may allocate fixed-width buffers, so oversized `VARCHAR(1000)` columns in `utf8mb4` make `GROUP BY`/`ORDER BY` spills more likely. Right-size columns.
- **Mismatched collations** cost CPU (per-row conversion) and, worse, index access. Check joins with `EXPLAIN` + `SHOW WARNINGS`.
- **Use ASCII for machine identifiers.** A `CHAR(36)` UUID or an ISO country code can be `CHARACTER SET ascii` (or `BINARY(16)` for UUIDs) to stay small regardless of the schema default.
- **Online conversion** of a large table needs disk for a full copy, replication-lag monitoring, and a cut-over plan; see the Online Schema Changes lesson in this topic.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SHOW CHARACTER SET; SHOW COLLATION LIKE 'utf8mb4%';` | List available charsets/collations | Choosing a target |
| `SHOW CREATE TABLE t;` | Shows table and per-column charset/collation | Auditing |
| `SET NAMES utf8mb4;` | Sets client/connection/results charset for the session | Scripts, clients |
| `ALTER TABLE t CONVERT TO CHARACTER SET utf8mb4 COLLATE …;` | Converts existing data and column definitions (rebuild) | The migration itself |
| `ALTER TABLE t DEFAULT CHARACTER SET utf8mb4;` | Changes only the default for *future* columns | After conversion |
| `col COLLATE utf8mb4_bin` / `BINARY col` | Per-expression collation override | Case-sensitive lookups (defeats a `_ci` index unless a matching index exists) |
| `CHAR_LENGTH(x)`, `LENGTH(x)`, `HEX(x)`, `CONVERT(x USING …)` | Inspect characters, bytes, raw bytes, convert | Debugging mojibake |
| `character-set-server`, `collation-server` | Server defaults | New databases |
| `information_schema.columns / tables` | Inventory of charsets | Pre-flight checks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `utf8mb4_0900_ai_ci` — modern default, multilingual user text | You need exact-case identifiers (use `_bin` or `_as_cs`) |
| `utf8mb4_bin` — tokens, hashes, usernames compared exactly | Human-facing sort order matters (binary order is code-point order) |
| `ascii`/`latin1` for known-ASCII codes | Data may ever contain non-ASCII; later conversion is another rebuild |
| Prefix index for long strings | You need uniqueness of the whole value or a covering index |
| In-place `CONVERT` on a small table | Large hot table — use gh-ost / pt-osc with replica lag throttling |

### 9. Common mistakes
- **Using `utf8` and assuming it is full UTF-8** → emoji rejected or truncated → use `utf8mb4` everywhere.
- **Changing only the table default** (`ALTER TABLE … DEFAULT CHARACTER SET`) → existing columns keep the old charset → use `CONVERT TO`.
- **Converting a column that actually holds UTF-8 bytes labelled `latin1`** → double-encoded mojibake → convert via `BINARY`/`VARBINARY` (`MODIFY col VARBINARY(n)` then `MODIFY col VARCHAR(n) CHARACTER SET utf8mb4`) after verifying bytes.
- **Not updating the client charset** → new rows still arrive mangled → set it in the driver and verify with `HEX()`.
- **Ignoring index length** → `ERROR 1071 Specified key was too long` mid-migration → shorten columns, use prefix indexes, or check row format.
- **Mixed collations across FK/join columns** → implicit conversion, lost index, or "Illegal mix of collations" → standardise.
- **Assuming `=` is exact** → `'ABC' = 'abc'` (and with PAD SPACE `'ABC' = 'ABC '`) is true → use a binary collation when exactness is required.
- **Case-insensitive uniqueness expectations** → an email unique index under `_ci` does enforce case-insensitive uniqueness; under `_bin` it does not → pick deliberately.

### 10. 🎯 Interview kit
**30-second answer:** A charset is the byte encoding, a collation is the comparison/sort rule. MySQL's `utf8` is a 3-byte subset, so use `utf8mb4` for emoji and full Unicode. Migrating means `ALTER TABLE … CONVERT TO CHARACTER SET utf8mb4`, which rebuilds the table, and you must watch index key limits, unique-key collisions under the new collation, and client connection charsets.

**Structure a longer answer:** clarify (versions, size, traffic) → inventory & verify stored bytes → test on a clone (index length, duplicates, row size) → online rebuild with a tool and lag throttling → switch defaults and client config → verify with EXPLAIN and round-trip tests → rollback plan.

**Likely follow-ups:**
- *Why did my join stop using the index after the migration?* → One side is now `utf8mb4`, the other `utf8mb3`; the indexed side is converted. Check `SHOW WARNINGS`; migrate the partner column.
- *What is the difference between `utf8mb4_general_ci` and `utf8mb4_0900_ai_ci`?* → Older simplified weights, PAD SPACE vs Unicode 9.0 UCA, NO PAD.
- *How do you do a case-sensitive search on a `_ci` column?* → `WHERE col = 'x' COLLATE utf8mb4_0900_as_cs` (won't use the `_ci` index efficiently), or `BINARY`, or a generated column with its own index.
- *How do you index a very long URL column?* → Prefix index, or a hash column (`CHAR(64)` SHA-256, unique) for exact lookup.
- *`CHAR_LENGTH` vs `LENGTH`?* → characters vs bytes.

**Signals interviewers look for:** you know the `utf8` trap; you mention index-size limits and uniqueness collisions; you treat conversion as a rebuild with an online strategy; you remember the client-side charset.

### 11. Check yourself
- [ ] Explain why an emoji cannot be stored in `utf8mb3` and what MySQL does in strict vs non-strict mode.
- [ ] Compute the index bytes for `VARCHAR(255)` under `utf8mb3` and `utf8mb4`, and say when a composite key breaks 3,072.
- [ ] Write the query that finds rows that become duplicates under `utf8mb4_0900_ai_ci`.
- [ ] Use `EXPLAIN` and `SHOW WARNINGS` to prove a charset mismatch disabled an index.
- [ ] Describe an online plan for converting a 500 GB table, including replica lag and rollback.
- [ ] Explain PAD SPACE vs NO PAD with an example comparison.
<!-- explainer:end -->

## Implementations (tools)

- [MySQL](<../_index.md>)

## Practice questions

- [Basic] [What does the UPPER, LOWER, and TRIM string function each do?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33366_What_does_the_UPPER_LOWER_and_TRIM_string_function_each_do.md>)
- [Basic] [How would you use SUBSTRING, CONCAT, and TRIM to normalize and format a user's display name?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33075_How_would_you_use_SUBSTRING_CONCAT_and_TRIM_to_normalize_and.md>)
- [Basic] [Explain the difference between CHAR and VARCHAR and when you would pick each.](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33335_Explain_the_difference_between_CHAR_and_VARCHAR_and_when_you.md>)
- [Intermediate] [How would you use NULLIF to avoid a division-by-zero error in a computed ratio column?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33044_How_would_you_use_NULLIF_to_avoid_a_division-by-zero_error_i.md>)
- [Intermediate] [What is the difference between the CHAR_LENGTH and LENGTH string functions on a multibyte string?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33074_What_is_the_difference_between_the_CHAR_LENGTH_and_LENGTH_st.md>)
- [Intermediate] [What is the difference between the utf8 and utf8mb4 character sets and why does it matter for emojis?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33546_What_is_the_difference_between_the_utf8_and_utf8mb4_characte.md>)
- [Intermediate] [What does a collation control and how would you make a search case-insensitive?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33553_What_does_a_collation_control_and_how_would_you_make_a_searc.md>)
- [Intermediate] [A user's "unsaved changes" warning fires falsely because the API normalizes data differently than the client — how does MySQL's collation/whitespace/trailing-space handling contribute?](<../../../PRACTICE/TOOLS/MySQL/MySQL_Platform/Q-33303_A_user_s_unsaved_changes_warning_fires_falsely_because_the_A.md>)

## Related concepts

- [String Functions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>) *(SQL Functions)*
- [String Functions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/String_Functions.md>) *(SQL Fundamentals)*
- [Cleaning Text and Numeric Fields](<../../../11_AI_and_ML/ML_Foundations/Data_Preprocessing/Cleaning_Text_and_Numeric_Fields.md>) *(Data Preprocessing)*

## References

- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › CHAR and VARCHAR: Non-Unicode Type Characters
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Collation and Sorting Rules
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Common Mistakes and Pitfalls
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Practice Exercises
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Bad Characters › Chapter 8. Bad Characters
- *Fuzzy Data Matching with SQL* — Jim Lehmer — Bad Characters › COLLATE
- *100 SQL Server Mistakes and How to Avoid Them* — Peter A. Carter — Data types › 3.2 #7 Always using variable-length strings
- *Cassandra: The Definitive Guide* — Jeff Carpenter, Eben Hewitt — in the Cassandra 2.2 release. While enumerated types are common in many languages, there is no
- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Expanding on Data Type Concepts › Large-Value String Data Types (MAX)
- *Hadoop: The Definitive Guide* — Tom White — Hadoop IO › Text
- *Hadoop: The Definitive Guide* — Tom White — Hive › The default storage format: Delimited text

