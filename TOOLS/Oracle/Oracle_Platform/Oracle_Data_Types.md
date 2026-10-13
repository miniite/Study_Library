---
id: "TOOLS/Oracle/Oracle_Platform/Oracle_Data_Types"
level: 4
type: "concept"
topic: "TOOLS/Oracle/Oracle_Platform"
tools: ["Oracle"]
---

# Oracle Data Types

**L4 Advanced** · [↑ Oracle Platform](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Oracle's core scalar types are `VARCHAR2`, `NUMBER`, `DATE` and `TIMESTAMP`, and most day-to-day bugs come from three things: Oracle's own NULL rules (empty string = NULL), the fact that `DATE` carries a time, and implicit conversions that quietly disable indexes.

### 1. The big picture
Every column, variable and expression has a data type, and Oracle converts between types when they mismatch. Knowing the types, their storage, and the conversion rules is what separates queries that work from queries that work *and* use their indexes. This lesson covers the character, numeric and date/time types, `NULL` semantics, the everyday functions (`TO_CHAR`, `TO_DATE`, `NVL`, `CASE`…), and the performance traps such as implicit conversion and literals instead of bind variables.

### 2. Core concepts and vocabulary
- **`VARCHAR2(n [BYTE|CHAR])`** — variable-length string, stores only the characters used (+ length). Oracle's recommended type; `VARCHAR` is reserved for a future standard-conformant semantic and currently behaves like `VARCHAR2`, so always write `VARCHAR2`.
- **`CHAR(n)`** — fixed-length, blank-padded to `n`; comparisons use padded semantics. Use rarely (codes of exact length).
- **`NUMBER(p,s)`** — decimal number with precision `p` (total significant digits, max 38) and scale `s` (digits right of the point; negative scale rounds to the left). `NUMBER` alone is floating-decimal up to 38 digits. Storage is variable (1–22 bytes).
- **`DATE`** — 7 bytes: century, year, month, day, **hour, minute, second**. Despite the name it always has a time part (no fractions).
- **`TIMESTAMP[(p)]`** — fractional seconds up to 9 digits; `TIMESTAMP WITH TIME ZONE` stores an offset/region; `TIMESTAMP WITH LOCAL TIME ZONE` normalizes to DB time zone and displays in session zone.
- **`INTERVAL YEAR TO MONTH` / `INTERVAL DAY TO SECOND`** — durations.
- **`NULL`** — absence of a value; not equal to anything, including another NULL. In Oracle an **empty string `''` is NULL** (for `VARCHAR2`/`CHAR`).
- **Format mask** — pattern like `'YYYY-MM-DD HH24:MI:SS'` used by `TO_DATE/TO_CHAR`.
- **Implicit vs. explicit conversion** — automatic conversion by Oracle vs. conversion you write with `TO_*`/`CAST`.
- **Bind variable** — placeholder (`:id`) whose value is supplied at execute time, letting the same parsed statement be reused.

### 3. How it works, step by step
**Character types and comparison**
1. `VARCHAR2(10 CHAR)` allows 10 characters regardless of multibyte encoding; `VARCHAR2(10 BYTE)` allows 10 bytes (default depends on `NLS_LENGTH_SEMANTICS`).
2. Max length is 4000 bytes (32767 with `MAX_STRING_SIZE=EXTENDED`); use `CLOB` beyond that.
3. `CHAR` comparison pads the shorter value; `VARCHAR2` comparison is exact (non-padded) unless one side is `CHAR`.
4. `||` concatenates (NULLs are treated as empty strings — `'a' || NULL = 'a'`); `CONCAT(a,b)` takes exactly two arguments.

**Numbers**
1. `NUMBER(10,2)` holds up to 99,999,999.99; inserting 123456789.123 rounds to scale 2 and fails if digits left of the point exceed 8 (`ORA-01438`).
2. `BINARY_FLOAT/BINARY_DOUBLE` are IEEE types, faster but inexact; use `NUMBER` for money.
3. `TO_NUMBER('1,234.50','9G999D99')` converts using a mask.

**Dates**
1. `SYSDATE` = server date and time (DATE precision, seconds). `SYSTIMESTAMP` = timestamp with fractional seconds and time zone. `CURRENT_DATE`/`CURRENT_TIMESTAMP` use the session time zone.
2. `TO_DATE('2026-10-13 14:30','YYYY-MM-DD HH24:MI')` parses; `TO_CHAR(d,'DD-Mon-YYYY')` formats.
3. `TRUNC(d)` removes time (to midnight); `TRUNC(d,'MM')` first of month; `ROUND` rounds to nearest unit.
4. Date literal: `DATE '2026-10-13'` or `TIMESTAMP '2026-10-13 14:30:00'` — independent of `NLS_DATE_FORMAT`.

**NULL behaviour**
1. Comparisons (`=`, `<>`, `<`) with NULL yield *unknown*, which `WHERE` treats as not true. Use `IS NULL` / `IS NOT NULL`.
2. `NVL(a,b)` returns b if a is NULL; `NVL2(a,b,c)` returns b if a is not NULL else c; `COALESCE(a,b,c,…)` returns the first non-NULL and short-circuits (ANSI, preferred).
3. `NOT IN (subquery)` returns no rows if the subquery produces any NULL: use `NOT EXISTS`.
4. B-tree indexes do not contain entries where *all* indexed columns are NULL, so `WHERE col IS NULL` cannot use a single-column B-tree index (add a constant column or use a function-based index).

### 4. Worked example
```sql
-- Setup
CREATE TABLE invoice (
  id          NUMBER(12)       PRIMARY KEY,
  customer_cd VARCHAR2(10)     NOT NULL,     -- character code, e.g. 'C000123456'
  amount      NUMBER(12,2)     NOT NULL,
  issued_on   DATE             DEFAULT SYSDATE NOT NULL,
  paid_at     TIMESTAMP(6),
  note        VARCHAR2(200)
);
CREATE INDEX invoice_cust_ix ON invoice(customer_cd);

-- Conversions
SELECT TO_CHAR(issued_on, 'YYYY-MM-DD HH24:MI') AS issued_txt,
       TO_DATE('13/10/2026','DD/MM/YYYY')        AS parsed,
       NVL(note,'(none)')                         AS note_txt,
       COALESCE(paid_at, SYSTIMESTAMP)            AS pay_ts,
       CASE WHEN amount >= 1000 THEN 'LARGE'
            WHEN amount >= 100  THEN 'MEDIUM'
            ELSE 'SMALL' END                      AS size_band,
       DECODE(SIGN(amount-100),1,'ABOVE',0,'EQUAL','BELOW') AS decode_band
FROM invoice;

-- The implicit-conversion trap
SELECT * FROM invoice WHERE customer_cd = 123456;     -- column is VARCHAR2, literal is NUMBER
SELECT * FROM invoice WHERE customer_cd = '123456';   -- correct
```
In the first query Oracle must make both sides comparable and, by its rules, converts the **column** to a number: the plan has `filter(TO_NUMBER("CUSTOMER_CD")=123456)` and uses `TABLE ACCESS FULL` instead of an index range scan, and any non-numeric value in the column raises `ORA-01722: invalid number`. Check with `EXPLAIN PLAN FOR …; SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY);` and look at the *Predicate Information* section.

Sorting with NULLs: `ORDER BY paid_at DESC NULLS LAST`. In Oracle NULLs sort *high* by default (last in `ASC`, first in `DESC`).

### 5. Under the hood
- **NUMBER internals.** Stored as base-100 digits with an exponent byte; two decimal digits per byte plus a length byte. That is why `NUMBER(10)` and `NUMBER(18)` do not differ by fixed width and why arithmetic is slower than binary integers.
- **DATE internals.** 7 bytes (century+100, year+100, month, day, hour+1, minute+1, second+1). Arithmetic is on days (`+1` = one day, `+1/24` = hour).
- **Empty string = NULL.** This stems from Oracle's historical storage: zero-length values are not stored (a NULL trailing column takes no space, an inner NULL takes 1 byte). Consequence: `WHERE note = ''` never matches; `LENGTH('')` is NULL.
- **Implicit conversion rules.** Between a character and a number, Oracle converts the *character* operand to number; between character and date it converts the character using `NLS_DATE_FORMAT` — a session setting, so such code works for one user and fails for another. Rule: never rely on it.
- **Bind variables.** A literal-based statement `WHERE id = 5` and `WHERE id = 6` are two different SQL texts and each requires a **hard parse** (syntax, semantics, optimization, shared pool latches). A statement with `:id` is parsed once and reused (**soft parse** or cursor cache hit). `CURSOR_SHARING=FORCE` can rewrite literals, but fixing code is better. Downside of binds: *bind peeking* uses the first value to choose a plan; adaptive cursor sharing (11g+) partly addresses skew.
- **OR vs. index.** `WHERE a = 1 OR b = 2` can use `OR-expansion` (`CONCATENATION`) only if both columns are indexed and the optimizer judges it cheaper; rewriting with `UNION ALL` (adding `LNNVL(a=1)` to the second branch to avoid duplicates) gives control.

### 6. Performance and design implications
- Match literal and bind types to column types; use explicit `TO_NUMBER/TO_DATE` on the *value*, never on the column.
- Do not wrap indexed columns in functions (`TRUNC(issued_on)`, `UPPER(name)`); use range predicates, or create a function-based index for the exact expression.
- Choose the narrowest correct numeric precision; money = `NUMBER(p,2)`, ids = `NUMBER(p)` with p sized to the domain; not `FLOAT`.
- Use `VARCHAR2` sized to the real maximum; oversize does not waste disk but hurts memory for fetch buffers and tools.
- Store dates as `DATE`/`TIMESTAMP`, never as strings (can't be range-compared or validated).
- Use bind variables in OLTP code; shared-pool pressure and hard parse CPU show up in `V$SQL` (many near-identical statements), `V$SYSSTAT` (`parse count (hard)`).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `SYSDATE`, `SYSTIMESTAMP` | Current server date / timestamp with zone | Timestamping rows |
| `TO_DATE(str, mask)`, `TO_CHAR(d, mask)` | Parse / format dates | Input/output conversion |
| `TO_NUMBER(str, mask)` | Parse numbers | Cleaning text input |
| `CAST(x AS type)` | ANSI conversion | Portable SQL |
| `NVL`, `NVL2`, `COALESCE`, `NULLIF` | NULL handling | Defaults, avoiding divide-by-zero (`NULLIF(d,0)`) |
| `CASE`, `DECODE` | Conditional expression | `CASE` preferred (ANSI, works in PL/SQL, treats NULL via `IS NULL`) |
| `SUBSTR(s,start,len)`, `INSTR(s,sub,start,nth)` | Slice/search strings | Parsing codes |
| `LIKE 'A%'`, `'_x'` | Pattern match; `%` any length, `_` one char | Prefix searches use index; leading `%` does not |
| `ALTER SESSION SET NLS_DATE_FORMAT = '…'` | Session default date mask | Avoid; use explicit masks |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `VARCHAR2` | Never use `VARCHAR`; `CHAR` only for exact fixed codes |
| `DATE` | You need sub-second precision or time zones → `TIMESTAMP` |
| `TIMESTAMP WITH TIME ZONE` | Pure local business dates → plain `DATE` is simpler |
| `COALESCE` | Both arguments are expensive and you need the side effects of both evaluated (NVL evaluates both args; COALESCE short-circuits) |
| `CASE` | You need quick legacy code compatibility (`DECODE` treats NULL = NULL as match) |
| Bind variables | Highly skewed columns in reporting queries, where literals give better plans |

### 9. Common mistakes
- Comparing with `= NULL` → no rows → use `IS NULL`.
- Expecting `''` to be an empty non-NULL string → conditions evaluate to unknown → use NULL logic, or store a sentinel.
- `WHERE issued_on = '2026-10-13'` relying on `NLS_DATE_FORMAT` and ignoring the time part → wrong rows or errors → `issued_on >= DATE '2026-10-13' AND issued_on < DATE '2026-10-14'`.
- Comparing `VARCHAR2` column to a number → full scan and possible `ORA-01722` → compare to string.
- `NOT IN` with nullable subquery → empty result → `NOT EXISTS`.
- Concatenating user input into SQL → hard parses and SQL injection → binds.
- `ROUND(date)` assumed to keep time → it snaps to midnight (noon rounds up).

### 10. 🎯 Interview kit
**30-second answer:** "`VARCHAR2` for text, `NUMBER(p,s)` for numerics, `DATE` for date plus time to the second and `TIMESTAMP` for fractions/time zones. NULL is unknown, so use `IS NULL`; and Oracle treats an empty string as NULL. Mismatched types cause implicit conversions that can disable indexes, so I compare like with like and use bind variables."

**Structure a longer answer:** types and sizes → NULL semantics → conversion functions → implicit conversion example with plan → binds/hard parse.

**Likely follow-ups:**
- *`DATE` vs. `TIMESTAMP`?* → 7 bytes/second precision vs. fractional seconds and zone options.
- *`CHAR` vs. `VARCHAR2`?* → padded fixed vs. variable; comparison semantics differ.
- *`NVL` vs. `NVL2` vs. `COALESCE`?* → default / branch on null-ness / first non-null, short circuit.
- *`DECODE` vs. `CASE`?* → Oracle-only function with NULL-equal semantics vs. ANSI expression with searched conditions.
- *Why prefer bind variables?* → avoid hard parse, protect shared pool, prevent injection.
- *Why can an `OR` prevent index use?* → predicates on different columns can't use one range scan; rewrite with `UNION ALL`.

**Signals interviewers look for:** the empty-string-is-NULL fact, awareness that DATE has time, ability to explain implicit conversion in a plan.

### 11. Check yourself
- [ ] State what `SELECT 'a' || NULL FROM dual` returns and why.
- [ ] Rewrite `WHERE TRUNC(issued_on)=DATE '2026-10-13'` to be index-friendly.
- [ ] Explain precision and scale of `NUMBER(7,-2)`.
- [ ] Show how to read predicate information in `DBMS_XPLAN` to spot `TO_NUMBER(col)`.
- [ ] Sort a column with NULLs first in descending order.
- [ ] Explain soft vs. hard parse and where binds help.
<!-- explainer:end -->

## Implementations (tools)

- [Oracle](<../_index.md>)

## Practice questions

- [Basic] [How do you get the current date and time in Oracle using SYSDATE and SYSTIMESTAMP?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35097_How_do_you_get_the_current_date_and_time_in_Oracle_using_SYS.md>)
- [Basic] [How do you concatenate strings in Oracle using the || operator or CONCAT function?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35210_How_do_you_concatenate_strings_in_Oracle_using_the_operator_.md>)
- [Basic] [What is the difference between the CHAR and VARCHAR2 data types in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35219_What_is_the_difference_between_the_CHAR_and_VARCHAR2_data_ty.md>)
- [Basic] [What is the ORDER BY clause and how do you sort ascending and descending with NULLS FIRST or NULLS LAST in Oracle?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35291_What_is_the_ORDER_BY_clause_and_how_do_you_sort_ascending_an.md>)
- [Intermediate] [How do you convert a string to a date in Oracle using TO_DATE and format masks?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35212_How_do_you_convert_a_string_to_a_date_in_Oracle_using_TO_DAT.md>)
- [Intermediate] [Why does Oracle recommend VARCHAR2 over VARCHAR and what is the difference?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35230_Why_does_Oracle_recommend_VARCHAR2_over_VARCHAR_and_what_is_.md>)
- [Intermediate] [Why can't you use = or != to compare a value with NULL in Oracle and what should you use instead?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35290_Why_can_t_you_use_or_to_compare_a_value_with_NULL_in_Oracle_.md>)
- [Deep] [Why should you prefer bind variables over literals in high-frequency SQL and how does this reduce hard parsing?](<../../../PRACTICE/TOOLS/Oracle/Oracle_Platform/Q-35109_Why_should_you_prefer_bind_variables_over_literals_in_high-f.md>)

## Related concepts

- [String Functions](<../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/String_Functions.md>) *(SQL Functions)*
- [Dynamic SQL](<../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Dynamic_SQL.md>) *(DBMS Concepts)*
- [MySQL Query Basics](<../../MySQL/MySQL_Platform/MySQL_Query_Basics.md>) *(MySQL Platform)*

