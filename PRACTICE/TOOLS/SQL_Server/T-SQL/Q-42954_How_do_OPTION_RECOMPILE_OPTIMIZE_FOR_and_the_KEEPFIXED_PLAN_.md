---
type: "practice"
difficulty: "Deep"
---

# How do OPTION (RECOMPILE) OPTIMIZE FOR and the KEEPFIXED PLAN hint each address parameter sniffing and what are their trade-offs?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Filtered Indexes and Plan Cache](<../../../../TOOLS/SQL_Server/T-SQL/Filtered_Indexes_and_Plan_Cache.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To understand how OPTION (RECOMPILE), OPTIMIZE FOR, and KEEPFIXEDPLAN hints in T-SQL address parameter sniffing, we first need to grasp what parameter sniffing is and its implications on query performance. Parameter sniffing occurs when SQL Server uses the actual values of parameters passed during a query execution to create a query plan that might not be optimal for other parameter values.

TL;DR:

OPTION (RECOMPILE) forces SQL Server to recompile the query each time it is executed, which can prevent parameter sniffing issues but increases compilation overhead.
OPTIMIZE FOR helps in creating an execution plan based on a given value or range of values for parameters, mitigating some issues with parameter sniffing. However, it might not always generate the best plan and can lead to suboptimal performance if the chosen value is not representative of the actual query workload.
KEEPFIXEDPLAN ensures that SQL Server uses the same execution plan across multiple executions with different parameter values, reducing the impact of parameter sniffing but potentially leading to poor performance for queries whose plans are not well-tuned.

Let's dive deeper into each concept and their trade-offs:

Understanding Parameter Sniffing

When a query is executed, SQL Server creates an execution plan based on the first set of parameters. If these initial values do not accurately represent the typical workload, the generated plan might perform poorly for subsequent executions with different parameter values.

OPTION (RECOMPILE)

The OPTION (RECOMPILE) hint forces SQL Server to recompile and rematerialize the query each time it is executed. This ensures that the most up-to-date statistics are used during compilation, which can prevent the issues caused by stale plan caches due to parameter sniffing.

#### Pros
Ensures the execution plan is always tailored to the current workload.
Can lead to better performance if the query is sensitive to different parameter values.

#### Cons
Increases overhead because of the compilation cost every time the query runs.
May not be feasible for queries that are executed frequently, as it can impact system performance.
OPTIMIZE FOR

The OPTIMIZE FOR hint allows you to specify a value or range of values to use when generating an execution plan. This hints SQL Server to create a plan based on the provided values rather than the actual parameter values from the first execution.

Syntax:

SQL
SELECT * FROM Sales.SalesOrderHeader
WHERE CustomerID = @CustomerID -- OPTIMIZE FOR (@CustomerID = 12345);

#### Pros

Can improve performance for queries that are executed with specific values.
Reduces the impact of parameter sniffing because the plan is based on a pre-defined value, which can better represent typical query workloads.

#### Cons

The chosen value might not always be representative of the actual workload, leading to suboptimal performance in some cases.
It does not guarantee that the execution plan will perform well for other values.
KEEPFIXEDPLAN

The KEEPFIXED PLAN hint ensures that SQL Server reuses the same execution plan across multiple executions. This can mitigate parameter sniffing issues by ensuring a consistent plan is used, but it might lead to poor performance if the initial plan is not well-tuned.

Syntax:

SQL
SELECT * FROM Sales.SalesOrderHeader WITH (KEEPFIXED PLAN)
WHERE CustomerID = @CustomerID;

#### Pros

Ensures consistency in execution plans, reducing the impact of parameter sniffing.
Can improve performance if the initial plan is well-tuned for typical workload values.

#### Cons

The fixed plan might not be optimal for other parameter values, leading to suboptimal performance.

#### Trade-offs
and Best Practices

The choice between these hints depends on the specific use case. OPTION (RECOMPILE) is useful in scenarios where the query execution environment changes frequently, while OPTIMIZE FOR and KEEPFIXEDPLAN are better suited for cases with a more stable workload or when you want to ensure consistency.

In practice, it's often best to start without any hints and monitor performance. If parameter sniffing issues arise, consider using one of these hints judiciously based on the specific requirements and workloads.

#### Closing Summary

Understanding OPTION (RECOMPILE), OPTIMIZE FOR, and KEEPFIXEDPLAN can significantly improve query performance by addressing the challenges posed by parameter sniffing. Each hint has its strengths and weaknesses, making them suitable for different scenarios. Careful consideration of their implications is essential to avoid unintended performance penalties.

#### Common Mistakes
Overusing OPTION (RECOMPILE): This approach increases compilation overhead without guaranteeing better performance in all cases.
Choosing a suboptimal value with OPTIMIZE FOR: The chosen value might not accurately represent the query workload, leading to poor performance.
Lack of monitoring: Without adequate monitoring, it's easy to introduce suboptimal plans that negatively impact system performance.

**Code:**

```sql
-- Example using OPTION (RECOMPILE)
SELECT * FROM Sales.SalesOrderHeader
WHERE CustomerID = @CustomerID -- OPTION (RECOMPILE);

-- Example using OPTIMIZE FOR
SELECT * FROM Sales.SalesOrderHeader
WHERE CustomerID = 12345 -- OPTIMIZE FOR (@CustomerID = 12345);

-- Example using KEEPFIXEDPLAN
SELECT * FROM Sales.SalesOrderHeader WITH (KEEPFIXED PLAN)
WHERE CustomerID = @CustomerID;
```

