---
type: "practice"
difficulty: "Deep"
---

# Explain how you would use SET STATISTICS IO/TIME or EXPLAIN ANALYZE to compare logical reads before and after tuning a procedure.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Stored Procedure Performance and Design](<../../../../01_Databases_and_SQL/Database_Objects/Stored_Procedures/Stored_Procedure_Performance_and_Design.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When tuning a stored procedure, it's crucial to understand its performance characteristics. Tools like SET STATISTICS IO/TIME or EXPLAIN ANALYZE help us get insights into how the database engine processes our queries and where potential bottlenecks lie. This answer will explain how to use these tools effectively to compare logical reads before and after tuning a procedure, providing step-by-step reasoning from first principles.

To start, let’s clarify what we’re probing: SET STATISTICS IO/TIME or EXPLAIN ANALYZE are diagnostic tools used to monitor the execution of T-SQL statements. SET STATISTICS IO provides information about logical and physical reads, while SET STATISTICS TIME gives details on the time taken by each query operation. Together, they help us assess how our database operations perform.

We’ll assume we have a stored procedure that needs tuning. Our goal is to understand its current performance and make improvements based on that understanding.

Initial Setup and Baseline Measurement:

Enable SET STATISTICS IO and SET STATISTICS TIME before running the initial version of your stored procedure.
Execute the stored procedure as it would normally be used in production.
Review the output from these statistics to get a baseline understanding of how many logical reads are occurring.

Tuning Process:

Identify potential bottlenecks such as inefficient joins, poor indexing, or excessive data retrieval.
Make changes to the stored procedure based on best practices and database optimization techniques.

Re-run with Tuned Procedure:

Turn off SET STATISTICS IO/TIME before re-running your procedure.
Execute the tuned version of the stored procedure.
Capture the new statistics output to compare against the initial baseline.

Analysis and Comparison:

Compare the logical read counts from both executions.
Analyze other metrics like CPU time, elapsed time, etc., if available.
Determine whether the tuning has resulted in a reduction of logical reads and overall improvement in performance.

**Comparison:**

```
Metric	Initial Execution	Optimized Version
Logical Reads	1,234	987
CPU Time (ms)	500	400
```

**Code:**

```sql
-- Enable STATISTICS IO and TIME before running the initial version
SET STATISTICS IO ON;
SET STATISTICS TIME ON;

-- Execute the stored procedure
EXEC MyTunedProcedure;

-- Capture the output
```

**Example input / output:**

```text
Initial execution:
- Logical Reads (initial): 1,234
- CPU Time: 500 ms

Optimized version:
- Logical Reads (after tuning): 987
- CPU Time: 400 ms
```

**Flow:**

1. Enable STATISTICS IO/ TIME
2. Execute Initial SP
3. Review Statistics
4. Tune SP Based on Analysis
5. Re-run Tuned SP withoutSTATISTICS
6. Capture New Statistics
7. Analyze and Compare Metrics

