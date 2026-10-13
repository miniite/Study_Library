---
type: "practice"
difficulty: "Deep"
---

# A stored proc is fast for one parameter and slow for another. Explain parameter sniffing in SQL Server and how OPTIMIZE FOR UNKNOWN, RECOMPILE, or forced plans stabilize it.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Large Batch Updates](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Large_Batch_Updates.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Parameter sniffing in SQL Server can cause performance issues when a stored procedure is executed with different input parameters because the initial execution plan may not be optimal for all parameter values. This issue arises because SQL Server compiles and caches an execution plan based on the first set of input parameters it encounters, which might not be suitable for subsequent executions.

#### Triaging the Question

The core question revolves around understanding why a stored procedure can perform differently based on its input parameters and how to mitigate these performance issues. The interviewer is probing your knowledge of SQL Server's optimization strategy, particularly parameter sniffing, and your ability to suggest solutions like OPTIMIZE FOR UNKNOWN, RECOMPILE, or forced plans.

#### First-Principles Solution

Parameter Sniffing: When a stored procedure is first executed with specific input parameters, SQL Server generates an execution plan based on these values. This plan is then cached and reused for subsequent executions with the same parameter value. However, if the initial set of input parameters leads to a suboptimal execution plan (e.g., poor statistics or skewed data distribution), this plan may not perform well when other parameter values are used.

OPTIMIZE FOR UNKNOWN: This option forces SQL Server to ignore the actual input parameters and generate an execution plan based on average statistics, which can help in cases where parameter values significantly vary. By doing so, it aims to create a more generic plan that is less sensitive to the initial parameter sniffing.

RECOMPILE Option: When you enable RECOMPILE, SQL Server compiles a new execution plan each time the stored procedure is executed, regardless of whether the cached plan exists or not. This ensures that the most recent statistics and data distribution are considered in every execution, potentially improving performance for varying input parameters.

Forced Execution Plans: You can manually specify which plans to use by using the OPTION (RECOMPILE) clause or creating multiple stored procedures with different parameter types. This approach provides more control over which plans are used but requires careful management and testing.

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, parameter sniffing can lead to performance issues in SQL Server when the initial execution plan is not optimal for all input parameters. Solutions like OPTIMIZE FOR UNKNOWN, RECOMPILE, or forced plans help mitigate these issues by ensuring more dynamic and contextually appropriate plans are generated.

#### Common mistakes
include:

Over-relying on OPTIMIZE FOR UNKNOWN without considering its limitations (e.g., increased compile time).
Not re-evaluating the performance impact of RECOMPILE options, especially in high-volume or frequently used procedures.
Failing to thoroughly test forced execution plans before implementing them.

For follow-ups, you might be asked about specific scenarios where these solutions are most applicable, the potential trade-offs between them, and how to monitor and optimize stored procedure performance over time.

**Comparison:**

```
Parameter	Value
Skew	True
MostFreq	1234
Execution Plan Type	Impact
Caching Initial Plan	Performance variability based on initial input.
OPTIMIZE FOR UNKNOWN	Generic plan, may improve performance for varying inputs.
RECOMPILE	New plan each execution, increases compile time but ensures up-to-date statistics.
```

**Code:**

```sql
-- Example using OPTIMIZE FOR UNKNOWN
CREATE PROCEDURE MyProcedure @param INT = 0
AS
BEGIN
    SELECT * FROM MyTable WHERE SomeColumn = @param;
END

EXEC MyProcedure 1234; -- This will use the cached plan with actual parameter sniffing.

EXEC MyProcedure WITH (OPTIMIZE FOR (@param UNKNOWN)); -- Generates a more generic plan.
```

**Example input / output:**

```text
Input:
EXEC MyProcedure 1234;

Output (initial plan):
- Uses a specific index for the parameter value 1234.

Input:
EXEC MyProcedure 5678;

Output (with OPTIMIZE FOR UNKNOWN):
- Uses a more generic execution plan based on average statistics.
```

**Data:**

```json
{
    "MyTable": {
        "Columns": ["SomeColumn", "OtherColumn"],
        "DataDistribution": {"Skewed": true, "Statistics": {"MostFrequentValue": 1234}}
    }
}
```

**Flow:**

1. Initial Execution
2. Plan Caching
3. Yes
4. Is Plan Optimal?
5. Use Cached Plan

