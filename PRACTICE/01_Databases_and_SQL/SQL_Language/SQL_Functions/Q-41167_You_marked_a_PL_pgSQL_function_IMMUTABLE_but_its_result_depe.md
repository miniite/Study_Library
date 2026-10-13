---
type: "practice"
difficulty: "Deep"
---

# You marked a PL/pgSQL function IMMUTABLE but its result depends on a config setting and returns stale cached values. What went wrong and how do you fix the volatility labeling?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Scalar vs Table-valued Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Functions/Scalar_vs_Table-valued_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PostgreSQL, functions can be classified by their "volatility" (or "volatility level") to optimize query planning and execution plans. The IMMUTABLE label means that the function's result does not change given the same input parameters and remains consistent across multiple calls, which can enable significant performance optimizations such as caching results. However, if a function is marked IMMUTABLE but its output depends on external factors like configuration settings or cached values, it leads to incorrect query planning.

Let’s dive into what went wrong and how to fix the volatility labeling.

#### Triage

The core issue here is that marking a function as IMMUTABLE implies that the result will be the same for identical inputs. If this assumption doesn't hold true due to external factors, PostgreSQL may cache results incorrectly or perform unnecessary re-computations, leading to stale data and performance degradation.

#### Assumptions
The function is being used in a context where its output should not change once it has been computed.
There are configuration settings or cached values that affect the function's result, making the function’s result non-deterministic for given inputs.
Step-by-Step Solution

Identify Dependencies:

Determine which external factors influence the function's result.

Change Volatility Labeling:

Change the IMMUTABLE label to STABLE or VOLATILE.
STABLE: The function’s result will not change within a transaction but can vary across transactions.
VOLATILE: The function's result may differ even with the same inputs due to external factors.

Update Function Implementation:

Modify the function implementation to handle dependencies correctly, ensuring it reflects the new volatility level accurately.

Test and Validate:

Run tests to ensure that the function behaves as expected under different conditions.
Verify that caching is managed properly or that appropriate cache invalidation strategies are in place.

Optimize Query Planning:

Re-optimize queries using the updated function, taking into account the new volatility level and potential performance implications.

#### Closing Summary

By changing the IMMUTABLE label to a more appropriate one (STABLE or VOLATILE) based on the actual behavior of the function, we ensure that PostgreSQL's query planner uses the correct assumptions. This can lead to better performance through proper caching and reduced unnecessary re-computations.

#### Common Mistakes/Follow-Ups
Mistake: Failing to recognize when a function’s result should not be cached due to external dependencies.

#### Follow-Up
Always validate that the chosen volatility level accurately reflects the function's behavior. Consider using EXPLAIN plans and benchmarks to ensure expected performance characteristics.

**Code:**

```sql
-- Example of changing function volatility from IMMUTABLE to STABLE
CREATE OR REPLACE FUNCTION my_function(config_param int, input_value int)
RETURNS int AS $$
DECLARE
    result int;
BEGIN
    -- Logic that depends on config settings or cached values
    IF config_param = 1 THEN
        result := (SELECT value FROM cache_table WHERE key = input_value);
    ELSE
        result := input_value * 2;  -- Example logic
    END IF;

    RETURN result;
END;
$$ LANGUAGE plpgsql STABLE;  -- Change from IMMUTABLE to STABLE or VOLATILE based on actual behavior

-- Test the function
SELECT my_function(1, 5);  -- Expected to use cached value if present
SELECT my_function(2, 5);  -- Should recompute since config_param is different
```

**Example input / output:**

```text
-- Example input and output
-- Assuming cache_table has a row with key = 5 and value = 10

my_function(1, 5) -> 10 (cached)
my_function(2, 5) -> 10 (recomputed)
```

**Flow:**

1. Yes
2. No
3. STABLE
4. VOLATILE
5. Identify Dependency
6. Is Function Immutable?
7. Mark as IMMUTABLE
8. Is It STABLE or VOLATILE?
9. Mark as STABLE, handlecaching
10. Mark as VOLATILE, disablecaching

