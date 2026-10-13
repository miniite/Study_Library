---
type: "practice"
difficulty: "Deep"
---

# How would you design a SQL query that joins a prompt-versions table to an answers-log table to attribute a quality drop to a specific prompt change and its deploy time?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Experiment Metrics in SQL](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Experiment_Metrics_in_SQL.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we need to design a SQL query that effectively joins two tables: prompt_versions and answers_log. The goal is to identify a quality drop in user responses attributed to a specific prompt change and its deploy time. This involves understanding the structure of both tables, considering potential data fields, and crafting a query that can trace the changes over time.

The prompt_versions table likely contains columns such as version_id, prompt_text, deploy_time, and possibly status. The answers_log table may include fields like user_id, response_score, and a reference to the prompt version, usually via version_id.

Here’s how we can break down the problem:

Identify Relevant Columns: We need columns in both tables that allow us to match prompts with their responses.

#### Cons
ider Time Frames: The quality drop might be observed over different time periods, so we should account for temporal aspects.
Cross-Validation: Ensure that the data in answers_log corresponds correctly to changes made in prompt_versions.

**Code:**

```sql
-- Sample SQL Query Design

WITH version_changes AS (
    SELECT
        pv.version_id,
        pv.prompt_text,
        pv.deploy_time,
        LAG(pv.status) OVER (PARTITION BY pv.version_id ORDER BY pv.deploy_time) as previous_status,
        pv.status as current_status
    FROM prompt_versions pv
),
quality_drops AS (
    SELECT
        al.user_id,
        av.version_id,
        al.response_score AS pre_change_score,
        lag(al.response_score) OVER (PARTITION BY av.version_id ORDER BY al.timestamp) as post_change_score,
        version_changes.deploy_time as change_deploy_time
    FROM answers_log al
    JOIN version_changes vc ON al.version_id = vc.version_id AND al.timestamp > vc.change_deploy_time
)
SELECT
    version_id,
    prompt_text,
    deploy_time,
    previous_status,
    current_status,
    user_id,
    pre_change_score,
    post_change_score,
    change_deploy_time
FROM quality_drops;
```

**Example input / output:**

```text
Input:
Prompt Versions Table (prompt_versions)
+------------+-------------+---------------+----------+
| version_id | prompt_text | deploy_time   | status   |
+------------+-------------+---------------+----------+
| v1         | Prompt A    | 2023-10-01    | active   |
| v2         | Prompt B    | 2023-10-05    | inactive |
+------------+-------------+---------------+----------+

Answers Log Table (answers_log)
+---------+--------------+---------------------+------------+
| user_id | response_score| timestamp            | version_id |
+---------+--------------+---------------------+------------+
| 1       | 8            | 2023-10-02T14:00:00Z| v1         |
| 2       | 7            | 2023-10-05T15:00:00Z| v2         |
| 3       | 6            | 2023-10-08T16:00:00Z| v2         |
+---------+--------------+---------------------+------------+

Output:
+------------+-------------+---------------+----------+-------------+------------+--------------+---------------------+
| version_id | prompt_text | deploy_time   | previous_status | current_status | user_id    | pre_change_score | post_change_score  |
+------------+-------------+---------------+----------+-------------+------------+--------------+---------------------+
| v2         | Prompt B    | 2023-10-05    | inactive | active      | 3          | 6             | -                 |
+------------+-------------+---------------+----------+-------------+------------+--------------+---------------------+
```

**Data:**

```json
{
  "prompt_versions": {
    "version_id": ["v1", "v2"],
    "prompt_text": ["Prompt A", "Prompt B"],
    "deploy_time": ["2023-10-01", "2023-10-05"],
    "status": ["active", "inactive"]
  },
  "answers_log": {
    "user_id": [1, 2, 3],
    "response_score": [8, 7, 6],
    "timestamp": ["2023-10-02T14:00:00Z", "2023-10-05T15:00:00Z", "2023-10-08T16:00:00Z"],
    "version_id": ["v1", "v2", "v2"]
  }
}
```

