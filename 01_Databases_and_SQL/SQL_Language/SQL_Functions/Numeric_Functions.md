---
id: "01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Functions"
tools: []
---

# Numeric Functions

**L4 Advanced** · [↑ SQL Functions](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Numeric functions (`ROUND`, `CEILING`, `FLOOR`, `ABS`, `MOD`, `POWER`, `SQRT`, `SIGN`, …) reshape numbers in a query — and the details that bite are *which way ties and negatives go*, integer division, and exact-versus-floating-point types.

### 1. The big picture
After dates and strings, numeric functions are the third everyday toolbox. They are row-by-row (scalar) functions: one number in, one number out. They matter most in money, billing, bucketing and reporting, where "off by one cent" or "off by one bucket" is a real bug. Most are simple; the interview and production difficulty is in **edge behavior**:

- does `ROUND` go away from zero or to even on ties?
- does "truncate" differ from `FLOOR` for negative numbers?
- is `7/2` 3 or 3.5?
- is the value `DECIMAL` (exact) or `FLOAT` (approximate)?

This lesson is vendor-neutral with SQL Server examples from *Beginning T-SQL* where noted.

### 2. Core concepts and vocabulary
- **Scalar function** — returns one value per input row.
- **Exact numeric** — `INT`, `BIGINT`, `DECIMAL(p,s)`/`NUMERIC`, `MONEY`: values stored exactly.
- **Approximate numeric** — `FLOAT`, `REAL`/`DOUBLE`: binary floating point; `0.1 + 0.2` ≠ `0.3` exactly.
- **Precision / scale** — total digits / digits after the decimal point.
- **Rounding** — replace a value with the nearest value at the given precision.
- **Truncation** — chop digits without rounding (toward zero).
- **`CEILING`/`CEIL`** — smallest integer ≥ x. **`FLOOR`** — largest integer ≤ x.
- **Modulo (`MOD`, `%`)** — remainder after division.
- **Integer division** — dividing two integers gives an integer in most engines (T-SQL, PostgreSQL); MySQL `/` gives a decimal and `DIV` gives integer.

### 3. How it works, step by step
**`ROUND(number, length [, function])`** (T-SQL signature from *Beginning T-SQL*):

1. `length` > 0 rounds to that many places on the right of the decimal point; `length` < 0 rounds to the left of it (tens, hundreds…).
2. The optional third argument, if non-zero, makes it **truncate** instead of round.
3. The result keeps the input type.

```sql
SELECT ROUND(1234.1294, 2)      AS two_right,    -- 1234.1300
       ROUND(1234.1294, -2)     AS two_left,     -- 1200.0000
       ROUND(1234.1294, 2, 1)   AS trunc_2,      -- 1234.1200
       ROUND(1234.1294, -2, 1)  AS trunc_neg2;   -- 1200.0000
```

Other everyday functions:

| Function | Meaning | Example |
|---|---|---|
| `ABS(x)` | Absolute value | `ABS(-4.2)` → 4.2 |
| `SIGN(x)` | -1, 0 or 1 | `SIGN(-9)` → -1 |
| `CEILING(x)` / `CEIL(x)` | Round up to integer | `CEILING(3.2)` → 4; `CEILING(-3.2)` → -3 |
| `FLOOR(x)` | Round down to integer | `FLOOR(3.8)` → 3; `FLOOR(-3.2)` → -4 |
| `MOD(a,b)` / `a % b` | Remainder | `MOD(10,3)` → 1 |
| `POWER(x,y)` | x to the power y | `POWER(5,2)` → 25 |
| `SQRT(x)` | Square root | `SQRT(16)` → 4; negative input is an error (or NULL in some engines) |
| `LOG`, `EXP`, `PI`, `RAND` | Math library | analytics |
| `GREATEST` / `LEAST` | Largest/smallest of several columns | see NULL lesson |
| `TRUNC(x[,d])` / `TRUNCATE(x,d)` | Chop digits | `TRUNC(-3.6)` → -3 |

### 4. Worked example
**Round down to the nearest 10 or 100 (bucket):**
```sql
SELECT FLOOR(n / 10.0)  * 10  AS bucket_10,
       FLOOR(n / 100.0) * 100 AS bucket_100
FROM (VALUES (47), (123), (-5)) v(n);
-- 47 -> 40 / 0,   123 -> 120 / 100,   -5 -> -10 / -100
```
Use `10.0`, not `10`: in T-SQL, `47/10` is integer division = 4 (which then `FLOOR`s to 4, giving 40 by luck). The decimal literal forces decimal division so `FLOOR` has a fraction to act on. For **negative** numbers, `FLOOR(-5/10.0) = FLOOR(-0.5) = -1`, giving -10; integer division would give 0 (truncation toward zero in T-SQL). That's the difference between "FLOOR of a division" and "integer division".

**Billing drift from rounding at different stages:**
```sql
-- 3 lines of 0.125 each
SELECT ROUND(0.125, 2) * 3     AS round_each,     -- 0.13 * 3 = 0.39
       ROUND(0.125 * 3, 2)     AS round_total;    -- ROUND(0.375,2) = 0.38
```
The two results differ by one cent. Decide *once* where rounding happens (per line, or on the invoice total), document it, and apply it consistently in the app, the database and the reports.

**Truncate vs round with negatives:**
```sql
SELECT ROUND(-3.6, 0)  AS r,   -- -4   (nearest)
       ROUND(-3.6, 0, 1) AS t, -- -3   (toward zero)
       FLOOR(-3.6)     AS f,   -- -4   (toward -infinity)
       CEILING(-3.6)   AS c;   -- -3
```
A billing routine that "truncated" in one place and "floored" in another gave different charges only for credit notes (negative amounts) — exactly the case tests rarely cover.

### 5. Under the hood
- **Tie-breaking.** SQL Server `ROUND` and PostgreSQL `round(numeric)` use **round half away from zero** (2.5 → 3, -2.5 → -3) for exact types. PostgreSQL `round(double precision)` follows the C library: **round half to even** on many platforms (2.5 → 2). Oracle `ROUND` is half away from zero; Python/`DECIMAL` default in some languages is half-even. "Banker's rounding" (half to even) reduces cumulative bias over large sums, which is why finance systems sometimes require it — implement it explicitly if needed.
- **Binary floating point.** `FLOAT` cannot represent 0.1 exactly; sums accumulate error and `ROUND(2.675, 2)` can give 2.67 for a double that is really 2.67499999… Use `DECIMAL(p,s)` for money; keep `FLOAT` for measurements and statistics.
- **Type of result.** `DECIMAL` arithmetic gets new precision/scale (e.g. `DECIMAL(10,2) * DECIMAL(10,2)` → `DECIMAL(21,4)`); once total precision passes the engine limit (38 in SQL Server), scale is reduced and you can lose decimals silently. Cast intermediate results deliberately.
- **Integer overflow.** `INT` arithmetic overflows at ±2,147,483,647 with an error; cast an operand to `BIGINT` first.
- **Modulo of negatives.** `-7 % 3` is -1 in T-SQL, PostgreSQL, MySQL, Java; `MOD(-7,3)` is -1 in Oracle too, but a mathematical modulus would be 2. Use `((a % n) + n) % n` for a non-negative result (e.g. hash bucketing).
- **Domain errors.** `SQRT(-1)` and `LOG(0)` raise errors in SQL Server/PostgreSQL; `MySQL` returns NULL. Guard with `CASE WHEN x >= 0 THEN SQRT(x) END`.

### 6. Performance and design implications
- Numeric functions are CPU-cheap, but wrapping an **indexed column** (`WHERE ROUND(price,0) = 10`) prevents an index seek. Rewrite as a range: `price >= 9.5 AND price < 10.5`.
- Do rounding **late** (at presentation or final aggregate) unless the business rule says otherwise; early rounding compounds error.
- Store money in `DECIMAL(19,4)` or integer minor units (cents), not `FLOAT`.
- Persisted computed columns with deterministic numeric functions can be indexed (e.g. `FLOOR(amount/100)` as a bucket) — handy for bucketed grouping.
- `RAND()` is nondeterministic and per-statement (same value for all rows in some engines); use `NEWID()`/`random()` per row where needed.

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `ROUND(x, n)` | Nearest at n places | Display, final billing |
| `ROUND(x, n, 1)` (T-SQL) / `TRUNC` / `TRUNCATE` | Chop toward zero | Rules that never round up |
| `FLOOR`, `CEILING` | Integer down / up | Bucketing, pagination (`CEILING(rows/page_size)`) |
| `%` / `MOD` | Remainder | Even/odd, round-robin, ring buffers |
| `ABS`, `SIGN` | Magnitude / direction | Differences, variance, direction flags |
| `CAST(x AS DECIMAL(p,s))` | Control exactness | Money, ratios |
| `ARITHABORT`, `ANSI_WARNINGS` (SQL Server) | Behaviour on overflow/divide-by-zero | Know defaults |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `DECIMAL` for money/rates | `FLOAT` for accounting totals |
| `ROUND` for display | Rounding intermediate values you'll sum later |
| `TRUNC` where policy says "never round up" (e.g. fee floors) | Negative amounts without deciding direction |
| Banker's rounding for large aggregated sets | Where regulations specify half-up |
| `FLOOR(x/n)*n` bucketing | Integer-only operands with negatives (silently truncates toward zero) |

### 9. Common mistakes
- Integer division (`7/2 = 3`) when you wanted 3.5 → wrong ratios → cast one operand to `DECIMAL` or multiply by `1.0`.
- Rounding in each line and again in the total → one-cent drift → single rounding policy.
- Using `FLOAT` for currency → penny errors that appear in reconciliation → `DECIMAL`.
- Confusing `TRUNCATE TABLE` with `TRUNCATE(x,d)` (MySQL) → one function, one DDL command with the same name → know context.
- `SQRT`/`LOG` on negatives or zero → run-time error mid-batch → guard.
- `ROUND` or `FLOOR` on indexed column in `WHERE` → scan → range predicate.
- Assuming `%` always gives non-negative → negative bucket ids → normalise.

### 10. 🎯 Interview kit
**30-second answer:** "`ROUND(x, n)` rounds to n decimal places (negative n rounds left of the point); `FLOOR`/`CEILING` go down/up to integers, `TRUNC` chops toward zero, `MOD` gives the remainder. The pitfalls are negative numbers, integer division, floating point, and rounding at inconsistent stages."

**Structure a longer answer:** list the family → clarify exact vs approximate types → show a negative-number example → explain how you'd keep a financial total consistent.

**Likely follow-ups:**
- *`CEIL` vs `FLOOR`?* → up vs down; for negatives `CEIL(-3.2) = -3`, `FLOOR(-3.2) = -4`.
- *`ROUND` vs `TRUNCATE`?* → nearest vs chop toward zero.
- *Round down to nearest 10?* → `FLOOR(x/10.0)*10`.
- *`FLOOR(a/b)` vs integer division?* → differ for negatives (floor vs toward zero).
- *Banker's vs arithmetic rounding?* → half-to-even vs half-away-from-zero; affects accumulated totals.
- *`SQRT` of a negative?* → error or NULL by engine.

**Signals interviewers look for:** you choose types deliberately (`DECIMAL`), define the rounding point, and test negatives.

### 11. Check yourself
- [ ] Predict `ROUND(1234.1294,-2)`, `ROUND(1234.1294,2,1)`, `FLOOR(-3.2)`, `-7 % 3`.
- [ ] Explain why `ROUND(price,0)=10` is slow and how to fix it.
- [ ] Show the one-cent drift of rounding per line vs total.
- [ ] Bucket values to the nearest lower 100, including negatives.
- [ ] Say when you'd use `FLOAT` and when `DECIMAL`.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the purpose of the ROUND function and how do you round to two decimal places?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41252_What_is_the_purpose_of_the_ROUND_function_and_how_do_you_rou.md>)
- [Basic] [Explain the difference between CEIL (CEILING) and FLOOR functions.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41264_Explain_the_difference_between_CEIL_CEILING_and_FLOOR_functi.md>)
- [Basic] [How does the MOD function (or % operator) work and what does it compute?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41267_How_does_the_MOD_function_or_operator_work_and_what_does_it_.md>)
- [Intermediate] [What is the difference between the ROUND and TRUNCATE functions when reducing decimal places?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41270_What_is_the_difference_between_the_ROUND_and_TRUNCATE_functi.md>)
- [Intermediate] [How do you round a number down to the nearest 10 or 100 using math functions?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41334_How_do_you_round_a_number_down_to_the_nearest_10_or_100_usin.md>)
- [Deep] [What is the difference between the FLOOR of a division and integer division in SQL?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41338_What_is_the_difference_between_the_FLOOR_of_a_division_and_i.md>)
- [Deep] [A financial report has rounding drift because ROUND is applied at different stages. Explain banker's rounding versus arithmetic rounding and how to keep totals consistent.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41203_A_financial_report_has_rounding_drift_because_ROUND_is_appli.md>)
- [Deep] [Explain how TRUNCATE versus ROUND interact with negative numbers and how a mismatch caused an off-by-one in a billing calculation.](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Functions/Q-41205_Explain_how_TRUNCATE_versus_ROUND_interact_with_negative_num.md>)

## Related concepts

- [Storing Money](<../SQL_Fundamentals/Storing_Money.md>) *(SQL Fundamentals)*
- [Date Arithmetic](<../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>) *(MySQL Platform)*
- [FIRST_VALUE and LAST_VALUE](<../../../TOOLS/PostgreSQL/PostgreSQL_Platform/FIRST_VALUE_and_LAST_VALUE.md>) *(PostgreSQL Platform)*

## References

- *Beginning T-SQL* — Kathi Kellenberger, Lee Everest — Using Mathematical Functions › ROUND

