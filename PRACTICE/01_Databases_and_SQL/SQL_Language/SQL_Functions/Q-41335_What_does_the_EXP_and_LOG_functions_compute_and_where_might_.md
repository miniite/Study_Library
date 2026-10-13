---
type: "practice"
difficulty: "Basic"
---

# What does the EXP and LOG functions compute and where might they be used?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1, SDE-1/2 · [↑ Scalar vs Table-valued Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Scalar_vs_Table-valued_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: The EXP function computes Euler's number ( e ) raised to a given power, while the LOG function returns the natural logarithm (base ( e )) of a value. These functions are essential in database queries for calculations involving exponential growth or decay, statistical analysis, and more.

#### Triage
The question asks about the functionality and usage scenarios of SQL’s EXP and LOG functions. We need to understand what these functions do and where they can be particularly useful in database operations.

Step-by-Step Solution:

Understanding EXP Function:

Definition: The EXP(x) function computes ( e^x ), where ( e ) is the base of the natural logarithm, approximately equal to 2.71828.
Use Case Examples:
Exponential Growth Modeling: In financial applications, you might use EXP to model compound interest or population growth over time.
Statistical Analysis: It’s used in statistical models like the Poisson distribution.

Understanding LOG Function:

Definition: The LOG(x) function returns the natural logarithm of ( x ), which is the power to which the base ( e ) must be raised to produce ( x ).
Use Case Examples:
Decay Modeling: Similar to exponential growth, but in reverse. For example, radioactive decay can be modeled using LOG.
Normalization of Data: In data preprocessing, logarithmic scaling is often used to normalize or stabilize the variance of a dataset.

Combining EXP and LOG:

EXP and LOG are inverses of each other. For example, EXP(LOG(x)) = x for ( x > 0 ). This property makes them useful in transforming data between linear and logarithmic scales.
They can also be used to solve equations involving exponential or logarithmic functions.

#### Example
Queries:

Suppose you have a table of population growth rates stored in a column named growth_rate:
SQL
SELECT EXP(growth_rate) AS future_population FROM populations;
To find the time it takes for an investment to double (assuming continuous compounding):
SQL
SELECT LOG(2) / interest_rate AS doubling_time FROM investments;

#### Closing Summary
The EXP and LOG functions are fundamental in SQL for handling exponential and logarithmic calculations. They are used extensively in financial modeling, statistical analysis, and data normalization tasks.

#### Common Mistakes/Follow-Ups

Mistake 1: Not understanding the exact definitions of EXP and LOG. Remember that LOG computes ( \ln(x) ), not base 10 logarithms.

#### Follow-Up
How would you handle negative values or zero in your calculations involving EXP and LOG? For instance, what happens if a growth rate is negative?
Mistake 2: Using these functions without considering the context. Always ensure that the domain of ( x ) (for LOG) and the power (for EXP) are appropriate for your use case.

#### Follow-Up
Can you provide an example where using EXP or LOG would lead to incorrect results if not properly handled?

**Comparison:**

| SQL Function | Definition                         | Use Case Examples                                      |
|--------------|-----------------------------------|-------------------------------------------------------|
| EXP          | \( e^x \) where \( e \approx 2.71828 \) | Exponential growth, financial modeling                 |
| LOG          | Natural logarithm of \( x \)       | Decay modeling, data normalization                     |

**Example input / output:**

```text
-- Input and expected output for EXP function
Input: SELECT EXP(growth_rate) FROM populations WHERE growth_rate = 0.1;
Expected Output:
+----------------+
| future_population |
+----------------+
|                1.1051709180756474 |
+----------------+

-- Input and expected output for LOG function
Input: SELECT LOG(2) FROM investments WHERE investment_value = 2 * initial_investment;
Expected Output:
+--------------------------------+
| doubling_time                  |
+--------------------------------+
|                             0.6931471805599453 |
+--------------------------------+
```

**Flow:**

1. EXP Function
2. Computes e^x
3. Use in Exponential GrowthModels
4. LOG Function
5. Returns ln(x)
6. Use in Decay Modeling, DataNormalization

