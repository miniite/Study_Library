---
id: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Storing_Money"
level: 4
type: "concept"
topic: "01_Databases_and_SQL/SQL_Language/SQL_Fundamentals"
tools: []
---

# Storing Money

**L4 Advanced** · [↑ SQL Fundamentals](<_index.md>)

<!-- explainer:start -->
## 📘 Lesson

> **In one line:** Store money in an exact numeric type (`DECIMAL/NUMERIC` with a fixed scale, or integer minor units like cents), never in `FLOAT/REAL`, and decide explicitly where and how rounding happens.

### 1. The big picture
Computers store `FLOAT`/`DOUBLE` as binary fractions. Most decimal fractions (0.1, 0.01, 19.99) have no exact binary representation, so values are stored as the nearest representable number and arithmetic accumulates tiny errors. In a balance sheet that shows up as sums that are off by a cent, reconciliations that never tie out, and totals that differ between an incremental job and a full recompute.

Exact numeric types store decimal digits (or integers), so `0.10 + 0.20 = 0.30` exactly. The cost is slightly more space and slower arithmetic than hardware floats, which is almost never a concern for financial data.

Related topics: choosing integer sizes for keys, the difference between `ROUND`, `FLOOR`, `CEILING` and truncation, integer division, and implicit conversion between numeric types.

### 2. Core concepts and vocabulary
- **Exact numeric** — `DECIMAL(p,s)` / `NUMERIC(p,s)`, `INT`, `BIGINT`, `SMALLINT`; values are exact.
- **Approximate numeric** — `FLOAT`, `REAL`, `DOUBLE PRECISION`; IEEE-754 binary floating point.
- **Precision (p)** — total number of significant digits; **scale (s)** — digits after the decimal point. `DECIMAL(10,2)` holds up to 99,999,999.99.
- **Minor units** — the smallest currency unit as an integer (cents, paise, yen); store 1999 for 19.99.
- **Rounding mode** — half-up (away from zero), half-even (banker's), floor, ceiling, truncate.
- **Integer division** — `INT / INT` returns an integer in many engines (`5/2 = 2`).
- **Currency code** — ISO 4217 (`USD`, `INR`, `JPY`); amounts are meaningless without it.
- **Allocation** — splitting an amount (e.g. 100.00 across 3 accounts) so parts sum exactly to the whole.

### 3. How it works, step by step
**Why floats drift.**
1. 0.1 in binary is 0.0001100110011… repeating, so it is stored as ≈ 0.1000000000000000055.
2. Adding 0.1 + 0.2 gives 0.30000000000000004.
3. Comparing with `= 0.3` is false; summing millions of such values in different orders gives slightly different totals (floating-point addition is not associative), so parallel aggregations are non-deterministic at the last digits.

**What exact types do.**
1. A `DECIMAL(12,2)` is stored as a scaled integer (SQL Server stores it as 5–17 bytes depending on precision; PostgreSQL `numeric` stores base-10000 digits).
2. Addition and subtraction are exact.
3. Multiplication and division produce a result type with its own precision and scale; if the result doesn't fit it is rounded or overflows depending on the engine and settings.

**Where rounding belongs.** Round at *business-defined boundaries* (per line, per invoice, per tax calculation as the law/contract requires), not repeatedly in intermediate steps, and store the rounded result.

### 4. Worked example
```sql
-- The bug
SELECT 0.1::double precision + 0.2::double precision = 0.3::double precision AS equal;  -- false
SELECT CAST(0.1 AS float) + CAST(0.2 AS float);                                         -- 0.30000000000000004 style

-- Exact
SELECT 0.1::numeric + 0.2::numeric = 0.3::numeric AS equal;                             -- true

-- Schema
CREATE TABLE payments (
  payment_id    bigint        PRIMARY KEY,
  account_id    bigint        NOT NULL,
  currency      char(3)       NOT NULL,                 -- ISO 4217
  amount        numeric(18,2) NOT NULL CHECK (amount <> 0),
  created_at    timestamptz   NOT NULL DEFAULT now()
);

-- Alternative: integer minor units (avoids scale questions in code and APIs)
CREATE TABLE payments2 (
  payment_id   bigint PRIMARY KEY,
  currency     char(3) NOT NULL,
  amount_minor bigint  NOT NULL        -- 1999 = 19.99 USD; JPY has 0 decimals
);

-- Integer division trap
SELECT 5/2        AS int_div,       -- 2
       5/2.0      AS dec_div,       -- 2.5
       CAST(5 AS numeric)/2 AS also_dec;

-- Allocate 100.00 into 3 parts that sum exactly
WITH parts AS (
  SELECT g AS n,
         trunc(100.00/3, 2)                                AS base
  FROM generate_series(1,3) g
)
SELECT n,
       base + CASE WHEN n = 1 THEN 100.00 - 3*base ELSE 0 END AS share   -- remainder to first part
FROM parts;                                                     -- 33.34, 33.33, 33.33
```

Migration of a FLOAT money column on a big table without a long lock:
1. Add `amount_dec numeric(18,2) NULL`.
2. Dual-write from the application (or a trigger) so new rows fill both.
3. Backfill in batches: `UPDATE … SET amount_dec = ROUND(amount_float::numeric, 2) WHERE id BETWEEN … AND amount_dec IS NULL`.
4. Verify with a reconciliation query (`SUM`, count of mismatches).
5. Switch readers, then drop the float column.

### 5. Under the hood
- **IEEE-754 double:** 1 sign bit, 11 exponent bits, 52 mantissa bits → about 15–17 significant decimal digits; `REAL` (24-bit mantissa) about 7. Integers up to 2^53 are exact, but fractions aren't.
- **SQL Server decimal arithmetic:** result precision/scale follow rules, e.g. for `a * b` precision = p1+p2+1, scale = s1+s2, capped at 38; when capped, the scale is reduced (minimum 6) and low digits are rounded, which can surprise in long multiplication chains. `DECIMAL` without arguments means `DECIMAL(18,0)`: a classic bug when someone writes `CAST(x AS DECIMAL)` and loses the fraction.
- **Rounding differences:** `ROUND` on `NUMERIC` rounds half away from zero in SQL Server and PostgreSQL; PostgreSQL `ROUND(double precision)` follows the C library and rounds half to even; Oracle `ROUND` is half away from zero; Python's `round()` is half-even. Banker's rounding reduces bias across many values and is required by some financial standards.
- **`MONEY` type:** SQL Server `MONEY` is a scaled 64-bit integer with 4 decimals; intermediate division in `MONEY` expressions truncates to 4 places, which can cause errors; PostgreSQL `money` depends on `lc_monetary` locale. Both are generally avoided in favor of `DECIMAL` + currency code.
- **Warehouses:** BigQuery `NUMERIC` (38,9) and `BIGNUMERIC`, Snowflake `NUMBER(p,s)` are exact; their `FLOAT` types are approximate.

### 6. Performance and design implications
- `DECIMAL(18,2)` costs 9 bytes in SQL Server; `BIGINT` minor units cost 8; both are negligible next to the correctness benefit.
- Aggregations on exact types are deterministic regardless of execution order or parallelism, so incremental and full backfills agree.
- Size scale for the *finest* unit you will ever handle (tax per share, crypto 8 decimals), not today's display needs; store more scale and round on presentation.
- `CHECK` constraints (non-negative balances, non-zero amounts) are cheap insurance.
- Keep the **currency next to the amount**; never sum across currencies without conversion.
- Document the rounding rule and test it with boundary values (0.005, 0.015, 2.675, negative values).

### 7. Commands and settings you should know
| Command / setting | What it does | When to use |
|---|---|---|
| `DECIMAL(p,s)` / `NUMERIC(p,s)` | Exact fixed-point | Money, rates, quantities |
| `ROUND(x, n)`, `FLOOR`, `CEILING`/`CEIL`, `TRUNC` | Rounding variants | Explicit business rounding |
| `CAST(x AS NUMERIC(p,s))` | Convert with specified scale | Controlling result scale |
| `CHECK (amount >= 0)` | Domain constraint | Prevent invalid values |
| `NUMERIC_ROUNDABORT` (SQL Server) | Error on precision loss | Strictness in financial code |
| Integer minor units | Avoid fractions | APIs, ledgers, payment providers |
| `SUM(amount)` per currency | Aggregate | Reporting |

### 8. Trade-offs and decisions
| Choose this when | Avoid / watch out when |
|---|---|
| `DECIMAL(p,s)` — readable, SQL-native | Application code converts to floats and loses exactness |
| Integer minor units — simple, language-safe | Currencies with differing exponents (JPY 0, KWD 3) — keep exponent in a currency table |
| Extra scale (4–8 places) for unit prices/FX | Displaying them unrounded to customers |
| Banker's rounding for large volumes | Regulation or contract demands half-up |
| `FLOAT` for measurements, ML features, scientific values | Anything that must reconcile to the cent |

### 9. Common mistakes
- `FLOAT` for amounts → cent-level mismatches in reconciliation → migrate to `DECIMAL`/minor units.
- Computing in `DECIMAL` but passing through a JSON/JS number → float again → send as string or minor units.
- `CAST(… AS DECIMAL)` with default `(18,0)` → fraction silently dropped → always specify precision and scale.
- Dividing integers → `5/2 = 2` → cast one operand to `NUMERIC`.
- Rounding each step of a long calculation → accumulated error → round once at the required boundary.
- Splitting a total evenly and rounding each part → parts don't sum to total → allocate the remainder deterministically.
- Storing amount as text → can't aggregate, sort wrongly ('100' < '20') → numeric type.

### 10. 🎯 Interview kit
**30-second answer:** Use exact numerics: `DECIMAL(p,s)` or integer minor units plus a currency code. Floats are binary approximations, so 0.1+0.2 ≠ 0.3 and sums depend on evaluation order. I round only at defined business boundaries, with a stated rounding mode, and allocate remainders so parts sum to totals.

**Structure a longer answer:** why floats fail (binary fractions, non-associativity) → exact options and sizing → rounding policy → conversion/migration plan → testing with edge values.

**Likely follow-ups:**
- *`DECIMAL` vs `NUMERIC`?* → synonyms in most engines.
- *Why do two jobs produce different sums?* → float non-associativity under parallelism.
- *How migrate a billion-row float column?* → add column, dual-write, batch backfill, reconcile, switch.
- *Which rounding?* → depends on spec; half-even reduces bias.

**Signals interviewers look for:** scale/precision fluency, rounding-mode awareness, currency handling, and a safe migration plan.

### 11. Check yourself
- [ ] Show with a query why `0.1 + 0.2 = 0.3` fails for floats.
- [ ] Choose precision and scale for a price, an FX rate and a share quantity.
- [ ] Write the batch backfill and reconciliation for a float → decimal migration.
- [ ] Allocate 0.10 among 3 payees without losing a cent.
- [ ] Explain why an incremental job and a full refresh disagree on float sums.
<!-- explainer:end -->

## Practice questions

- [Basic] [What is the difference between INT, BIGINT, and SMALLINT, and how do you choose the right one for an id column?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39799_What_is_the_difference_between_INT_BIGINT_and_SMALLINT_and_h.md>)
- [Basic] [How do you round a numeric column to two decimal places, and how is ROUND different from TRUNCATE of the decimals?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39843_How_do_you_round_a_numeric_column_to_two_decimal_places_and_.md>)
- [Basic] [What SQL data types would you use to store model features (e.g. INT vs FLOAT vs DECIMAL) and why does precision matter?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-40572_What_SQL_data_types_would_you_use_to_store_model_features_e_.md>)
- [Basic] [How would you store a metric value safely as a numeric type, and why is storing it as text a problem for aggregation?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-41048_How_would_you_store_a_metric_value_safely_as_a_numeric_type_.md>)
- [Intermediate] [What is the difference between the FLOAT, DECIMAL, and NUMERIC data types, and which is exact?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39798_What_is_the_difference_between_the_FLOAT_DECIMAL_and_NUMERIC.md>)
- [Intermediate] [What is integer division in SQL and why might SELECT 5/2 return 2 instead of 2.5?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39846_What_is_integer_division_in_SQL_and_why_might_SELECT_5_2_ret.md>)
- [Intermediate] [How would you round, floor, and ceil a numeric value in a query, and where do these matter for money?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39955_How_would_you_round_floor_and_ceil_a_numeric_value_in_a_quer.md>)
- [Deep] [Why is storing money as a floating-point type risky, and what data type should you use instead?](<../../../PRACTICE/01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Q-39967_Why_is_storing_money_as_a_floating-point_type_risky_and_what.md>)

## Related concepts

- [Numeric Functions](<../SQL_Functions/Numeric_Functions.md>) *(SQL Functions)*
- [Date Arithmetic](<../../../TOOLS/MySQL/MySQL_Platform/Date_Arithmetic.md>) *(MySQL Platform)*
- [Advanced Aggregation](<../Aggregation/Advanced_Aggregation.md>) *(Aggregation)*

## References

- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design
- *Grokking Relational Database Design* — Qiang Hao, Michail Tsikerdekis — Grokking Relational Database Design (cont.)
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Approximate Numeric Data Types
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Case Study: The Safety of Explicit Operations
- *Ultimate T-SQL for SQL Server* — Alula Mekonnen Kassa — Chapter 2: Data Types, NULLs and Conversions › Common Pitfalls in Conversions

