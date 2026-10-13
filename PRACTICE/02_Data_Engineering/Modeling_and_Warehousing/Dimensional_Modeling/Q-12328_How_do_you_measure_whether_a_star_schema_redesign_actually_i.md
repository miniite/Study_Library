---
type: "practice"
difficulty: "Intermediate"
---

# How do you measure whether a star schema redesign actually improved query performance rather than relying on intuition?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ OLTP vs Star Schema Performance](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/OLTP_vs_Star_Schema_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating if a star schema redesign has genuinely improved query performance, it's crucial to move beyond subjective measures and adopt objective metrics. This involves understanding the current baseline performance, setting up comprehensive monitoring during the redesign, and then analyzing post-redesign performance. The key steps include defining clear performance metrics, conducting controlled tests, and using appropriate tools for data analysis.

**Comparison:**

```
Query ID	Start Time	Execution Time (ms) Pre-Redesign	Execution Time (ms) Post-Redesign
123	2023-10-01 08:00	50	40
456	2023-10-01 09:00	45	35
```

**Code:**

```sql
-- Example SQL query to monitor performance changes
SELECT
    query_id,
    start_time,
    execution_time_ms,
    user_id,
    db_name
FROM
    performance_audit_log;
```

**Example input / output:**

```text
**Pre-Redesign Baseline Measurement**

Input: `SELECT execution_time_ms FROM performance_audit_log WHERE db_name = 'old_schema' AND start_time >= '2023-10-01'`

Output:
| query_id | start_time        | execution_time_ms |
|----------|-------------------|------------------|
| 123      | 2023-10-01 08:00  | 50                |
| 456      | 2023-10-01 09:00  | 45                |

**Post-Redesign Performance Analysis**

Input: `SELECT execution_time_ms FROM performance_audit_log WHERE db_name = 'new_schema' AND start_time >= '2023-11-01'`

Output:
| query_id | start_time        | execution_time_ms |
|----------|-------------------|------------------|
| 789      | 2023-11-01 08:00  | 40                |
| 901      | 2023-11-01 09:00  | 35                |
```

**Flow:**

1. Yes
2. No
3. Yes
4. Define Performance Metrics
5. Are Metrics Clear?
6. Set Up Monitoring Tools
7. Redesign Star Schema
8. Implement Changes?
9. Monitor Performance DuringTransition
10. Analyze Data After Redesign
11. Compare Post-RedesignMetrics with Baselines
12. Evaluate Improvement

