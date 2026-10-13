---
type: "practice"
difficulty: "Deep"
---

# Walk me through investigating a report where the same procedure sometimes returns in 200ms and sometimes in 20 seconds.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ MERGE Pitfalls and Error Handling](<../../../../TOOLS/SQL_Server/T-SQL/MERGE_Pitfalls_and_Error_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with performance variability in T-SQL stored procedures, the key is to systematically identify the root causes of the fluctuations. Typically, these could stem from changes in query execution plans, resource contention, or external factors like network latency. In this scenario, we'll investigate by following a methodical approach.

**Code:**

```sql
-- Example procedure call and trace setup
EXEC YourProcedure;

-- Enable SQL Profiler to capture detailed information about the procedure's execution
-- You can use T-SQL commands or SSMS for this step
```

**Example input / output:**

```text
Input (SQL Profiler trace):
- Start SQL Profiler to capture execution details of the procedure.

Output:
- Detailed trace data including start and end times, SPID, and executed statements.
```

**Data:**

```plaintext
-- Sample output from SQL Profiler
|EventClass|StartTime           |EndTime             |SPID|TextData                    |
|----------|--------------------|-------------------|----|----------------------------|
|SQLBatchCompleted|2023-10-05 14:30:00|2023-10-05 14:30:00|123 |SELECT * FROM SalesOrders   |
|RPCCompleted  |2023-10-05 14:31:00|2023-10-05 14:31:10|456 |EXEC YourProcedure          |
```

