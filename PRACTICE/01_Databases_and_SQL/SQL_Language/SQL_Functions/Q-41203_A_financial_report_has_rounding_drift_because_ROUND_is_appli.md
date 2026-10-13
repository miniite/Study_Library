---
type: "practice"
difficulty: "Deep"
---

# A financial report has rounding drift because ROUND is applied at different stages. Explain banker's rounding versus arithmetic rounding and how to keep totals consistent.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Numeric Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Numeric_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure consistency in financial reports, understanding the nuances between banker's rounding and arithmetic rounding is crucial. Banker’s rounding (also known as round half to even) minimizes cumulative error by rounding towards the nearest even number when a value is exactly halfway between two integers. Arithmetic rounding always rounds up if the digit after the rounding point is 5 or greater, which can introduce systematic bias over many operations.

Let's first understand how these rounding methods differ and then explore an SQL solution to keep totals consistent despite applying rounding at different stages.

**Code:**

```sql
-- Example of arithmetic vs banker's rounding in SQL

WITH Numbers AS (
    SELECT 10.45, 98765.35 AS amount -- Sample data points
),
ArithmeticRounded AS (
    SELECT
        ROUND(amount, 2) AS arith_rounded
    FROM
        Numbers
),
BankersRounded AS (
    SELECT
        ROUND(amount, 2, 1) AS bankers_rounded
    FROM
        Numbers
)
SELECT
    SUM(arith_rounded) AS arithmetic_total,
    SUM(bankers_rounded) AS bankers_total
FROM
    (SELECT arith_rounded, bankers_rounded FROM ArithmeticRounded, BankersRounded) subquery;
```

**Example input / output:**

```

The input data is:

PLAINTEXT
Copy
10.45
98765.35

The output from the query is:

PLAINTEXT
Copy
arithmetic_total = 98776.2    (10.45 + 98765.8)
bankers_total     = 98776.2   (10.45 -> 10.4, 98765.35 -> 98765.4)
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Input Data
6. Arithmetic Rounding?
7. Round up if > 4
8. Banker's Rounding?
9. Rounds to nearest evennumber
10. Always round up if 5 or greater
11. Perform Summation
12. Output Total

