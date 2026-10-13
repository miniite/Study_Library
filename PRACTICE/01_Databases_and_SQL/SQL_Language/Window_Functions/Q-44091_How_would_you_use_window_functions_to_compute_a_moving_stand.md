---
type: "practice"
difficulty: "Expert"
---

# How would you use window functions to compute a moving standard deviation, and why can STDDEV OVER be numerically unstable over long frames?

**Difficulty:** Expert · **Levels:** SDE-1, SDE-1/2 · [↑ Time-based Windows](<../../../../01_Databases_and_SQL/SQL_Language/Window_Functions/Time-based_Windows.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a moving standard deviation using SQL window functions, we'll leverage STDDEV with an OVER clause to slide over rows in a specified frame. However, this approach can be numerically unstable for long frames due to the way variance is calculated and aggregated.

Firstly, let's understand what a moving standard deviation is. It measures how spread out your data points are from their mean within a given window of rows. The STDDEV function computes the square root of the variance. When using it with an OVER clause, we're essentially calculating this for each row in a sliding window.

#### Triaging the Question

The question is probing:

Understanding Window Functions: How you can use STDDEV OVER to calculate moving standard deviations.
Numerical Stability Concerns: The inherent issues with using variance calculations over long windows, especially in SQL implementations.

We'll assume a common scenario where we have a dataset of time-series data stored in a table like this:

SQL
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL,
    value DECIMAL(10, 2) NOT NULL
);

#### First-Principles Solution

The standard way to calculate the moving standard deviation using STDDEV with an OVER clause is straightforward but can be numerically unstable over long windows. Here’s how it looks:

SQL
SELECT
    date,
    value,
    STDDEV(value) OVER (
        ORDER BY date
        ROWS BETWEEN 3 PRECEDING AND CURRENT ROW
    ) as moving_stddev
FROM sales;

In this query, ROWS BETWEEN 3 PRECEDED AND CURRENT ROW defines a window that includes the current row and the two preceding rows. The result is a computed column showing the standard deviation of the last four points.

Why STDDEV OVER Can Be Numerically Unstable

The instability arises from how SQL typically calculates variance (which is the square of standard deviation). Variance calculations can suffer from numerical issues because:

Variance Calculation: Variance sums up squared deviations from the mean, which means small values get amplified.
Precision Issues: As the window size grows, more rows are involved in the calculation, leading to potential loss of precision due to floating-point arithmetic.

For example, consider a simple dataset where all values are close but not identical:

PLAINTEXT
value: 100.01, 100.02, 100.03, 99.99, 100.00

The mean is 100.01, and the variance involves squaring differences from this mean, which can lead to small but significant inaccuracies.

Alternative Approaches

To mitigate numerical issues, you might consider:

Using VAR instead of STDDEV: Variance can sometimes be more stable since it is less sensitive to outliers.
Custom Aggregations: Implementing custom window functions or using a library that handles variance calculation with higher precision.

#### Closing Summary
and Common Mistakes

When implementing moving standard deviations:

Ensure you understand the underlying numerical issues, especially with long windows.

#### Cons
ider alternative methods like VAR for stability.
Always test thoroughly to ensure accuracy in your specific use case.

#### Common mistakes
include not fully understanding how variance calculations interact with window functions or assuming that the built-in SQL functions are always sufficient without considering edge cases and potential numerical instability.

#### Follow-Ups
How would you handle very large datasets where performance is a concern?
Are there any SQL features or libraries you could use to improve stability?

**Comparison:**

| date       | value | moving_stddev |
|------------|-------|---------------|
| 2023-10-01 | 100.01| NULL          |
| 2023-10-02 | 100.02| NULL          |
| 2023-10-03 | 100.03| NULL          |
| 2023-10-04 | 99.99 | 0.015811      |
| 2023-10-05 | 100.00| 0.015811      |

**Code:**

```sql
-- Example of moving standard deviation query
SELECT
    date,
    value,
    STDDEV(value) OVER (
        ORDER BY date
        ROWS BETWEEN 3 PRECEDING AND CURRENT ROW
    ) as moving_stddev
FROM sales;
```

**Example input / output:**

```

Sample input:

PLAINTEXT
Copy
date: 2023-10-01, value: 100.01
date: 2023-10-02, value: 100.02
date: 2023-10-03, value: 100.03
date: 2023-10-04, value: 99.99
date: 2023-10-05, value: 100.00

Sample output:

PLAINTEXT
Copy
date: 2023-10-01, moving_stddev: NULL (no window)
date: 2023-10-02, moving_stddev: NULL (no window)
date: 2023-10-03, moving_stddev: NULL (no window)
date: 2023-10-04, moving_stddev: 0.0158113883
date: 2023-10-05, moving_stddev: 0.0158113883
```

**Data:**

```plaintext
CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL,
    value DECIMAL(10, 2) NOT NULL
);
```

**Flow:**

1. Yes
2. No
3. Start
4. Is window size large?
5. Use variance instead of std dev
6. Handle precision issuescarefully
7. Standard STDDEV OVER is fine
8. Done

