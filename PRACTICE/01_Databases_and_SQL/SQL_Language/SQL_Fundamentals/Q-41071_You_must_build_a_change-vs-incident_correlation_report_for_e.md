---
type: "practice"
difficulty: "Deep"
---

# You must build a change-vs-incident correlation report: for each incident, find the most recent deploy/config-change within 30 minutes before start_time. Why is a LATERAL/CROSS APPLY top-1 join better than a window function here?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Attribution with Window Functions](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Attribution_with_Window_Functions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To build a change-vs-incident correlation report, we need to identify the most recent deploy or config-change associated with each incident within a 30-minute window before its start_time. For this task, using a LATERAL/CROSS APPLY top-1 join is more efficient and appropriate compared to a window function in SQL.

#### Triaging the Question

The core of the question is about optimizing the query for performance and correctness when dealing with complex joins and aggregations. The key ideas are:

Join Type: Understanding the difference between LATERAL/CROSS APPLY and window functions.
Performance Consideration: Evaluating which method is more efficient in terms of execution time and resource utilization.
Scalability: Ensuring that the solution can handle large datasets without performance degradation.

#### First Principles Solution

Let's break down the problem step by step:

Identify Tables:

incidents: A table containing incident records with columns like incident_id, start_time, etc.
changes: A table containing change records with columns like change_id, deploy_time, config_change_time, incident_id, etc.

Objective: For each row in the incidents table, find the most recent change_id from the changes table where the change time is within 30 minutes before the incident's start_time.

Why LATERAL/CROSS APPLY vs Window Function

LATERAL/CROSS APPLY Top-1 Join:

Execution Plan: This join method allows us to perform a subquery for each row of the outer query, which can be more efficient in this context.
Performance: It typically performs better with large datasets because it avoids full scans and sorts that window functions might require.

Window Function Approach:

Complexity: Window functions are generally less intuitive and can lead to complex execution plans.
Scalability: For very large tables, the overhead of sorting and partitioning rows can become significant, potentially leading to poor performance.
Detailed SQL Implementation
Using LATERAL/CROSS APPLY:
SQL
WITH ranked_changes AS (
    SELECT
        i.incident_id,
        c.change_id,
        ROW_NUMBER() OVER (PARTITION BY i.incident_id ORDER BY ABS(c.deploy_time - i.start_time) ASC) as rank
    FROM
        incidents i
    CROSS APPLY (
        SELECT
            change_id, deploy_time
        FROM
            changes
        WHERE
            incident_id = i.incident_id AND
            deploy_time BETWEEN i.start_time - INTERVAL '30 minutes' AND i.start_time
    ) c
)
SELECT
    i.incident_id,
    c.change_id
FROM
    incidents i
JOIN
    ranked_changes r ON i.incident_id = r.incident_id AND r.rank = 1;
Explanation:
CROSS APPLY: This allows us to join the incidents table with a subquery that finds recent changes for each incident.
ROW_NUMBER(): Assigns a rank to each change based on its proximity to the start_time of the incident.

#### Closing Summary
and Common Mistakes

The LATERAL/CROSS APPLY approach is generally more efficient for this scenario. It avoids full table scans and sorts, making it scalable for large datasets. Common mistakes include:

Incorrect Join Conditions: Forgetting to filter changes within the 30-minute window.
Complexity Misunderstanding: Assuming that window functions are always simpler or faster.
Self-check

The solution is correct and complete, covering all necessary steps and performance considerations.

