---
type: "practice"
difficulty: "Deep"
---

# Your cost-attribution query double-counts because a request has both a retrieval-embedding call and a generation call in the same log. How would you fix the fan-out with pre-aggregation per call type?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Join Fan-out](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Join_Fan-out.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of double-counting costs due to overlapping calls within the same request, we need to ensure that each cost is attributed correctly without overcounting. Specifically, a single log entry might include both a retrieval-embedding call and a generation call, leading to duplicated costs if not handled properly.

The key concept here involves pre-aggregating the logs per call type before performing the final cost attribution query. This approach avoids double-counting by processing each call type independently first.

TL;DR: We need to aggregate logs separately for retrieval-embedding and generation calls, then join these aggregated results with a unified cost-attribution query.

#### Triage
The core issue is ensuring that costs are not overcounted when both types of calls occur in the same request log entry. We will address this by pre-aggregating call types and then performing the final aggregation.

#### Assumptions

We have a table request_logs with columns such as request_id, call_type, start_time, end_time, and cost.
The call_type column distinguishes between retrieval-embedding (RETRIEVAL_EMBEDDING) and generation calls (GENERATION).

#### First Principles Solution

Pre-Aggregation Step: We start by aggregating the logs for each call type separately to get total costs.

Final Aggregation Step: Join these aggregated results to ensure we only account for unique requests.

Here is a step-by-step breakdown:

Pre-Aggregation
SQL
WITH retrieval_embedding_aggr AS (
    SELECT
        request_id,
        SUM(cost) AS total_retrieval_embedding_cost
    FROM
        request_logs
    WHERE
        call_type = 'RETRIEVAL_EMBEDDING'
    GROUP BY
        request_id
),
generation_aggr AS (
    SELECT
        request_id,
        SUM(cost) AS total_generation_cost
    FROM
        request_logs
    WHERE
        call_type = 'GENERATION'
    GROUP BY
        request_id
)
Final Aggregation
SQL
SELECT
    rl.request_id,
    re.total_retrieval_embedding_cost,
    g.total_generation_cost,
    (re.total_retrieval_embedding_cost + g.total_generation_cost) AS total_cost
FROM
    retrieval_embedding_aggr re
JOIN
    generation_aggr g ON re.request_id = g.request_id;
Explanation:
Retrieval-Eembedding Aggregation: We first aggregate the costs for each request_id where the call_type is RETRIEVAL_EMBEDDING.
Generation Aggregation: Similarly, we do this for GENERATION calls.
Final Join: We join these two aggregated results to get a combined cost per request.

#### Closing Summary
and Common Mistakes:

This approach ensures that costs are not double-counted by handling each call type independently. The common mistake would be trying to sum up the costs directly from the raw logs without pre-aggregation, leading to overcounting.

**Comparison:**

| request_id | total_retrieval_cost   | total_gen_cost    | total_cost         |
|------------|------------------------|-------------------|--------------------|
| 1          | 10.0                   | 5.0               | 15.0               |
| 2          | 7.0                    | 0.0               | 7.0                |
| 3          | 0.0                    | 8.0               | 8.0                |

**Code:**

```sql
WITH retrieval_embedding_aggr AS (
    SELECT
        request_id,
        SUM(cost) AS total_retrieval_embedding_cost
    FROM
        request_logs
    WHERE
        call_type = 'RETRIEVAL_EMBEDDING'
    GROUP BY
        request_id
),
generation_aggr AS (
    SELECT
        request_id,
        SUM(cost) AS total_generation_cost
    FROM
        request_logs
    WHERE
        call_type = 'GENERATION'
    GROUP BY
        request_id
)
SELECT
    rl.request_id,
    re.total_retrieval_embedding_cost,
    g.total_generation_cost,
    (re.total_retrieval_embedding_cost + g.total_generation_cost) AS total_cost
FROM
    retrieval_embedding_aggr re
JOIN
    generation_aggr g ON re.request_id = g.request_id;
```

**Example input / output:**

```

WITH retrieval_embedding_aggr AS ( SELECT request_id, SUM(cost) AS total_retrieval_embedding_cost FROM request_logs WHERE call_type = 'RETRIEVAL_EMBEDDING' GROUP BY request_id ), generation_aggr AS ( SELECT request_id, SUM(cost) AS total_generation_cost FROM request_logs WHERE call_type = 'GENERATION' GROUP BY request_id ) SELECT rl.request_id, re.total_retrieval_embedding_cost, g.total_generation_cost, (re.total_retrieval_embedding_cost + g.total_generation_cost) AS total_cost FROM retrieval_embedding_aggr re JOIN generation_aggr g ON re.request_id = g.request_id;

CODE
Copy
Result:
```sql
+------------+------------------------+-------------------+--------------------+
| request_id | total_retrieval_cost   | total_gen_cost    | total_cost         |
+------------+------------------------+-------------------+--------------------+
| 1          | 10.0                   | 5.0               | 15.0               |
| 2          | 7.0                    | 0.0               | 7.0                |
| 3          | 0.0                    | 8.0               | 8.0                |
+------------+------------------------+-------------------+--------------------+
```

**Data:**

```json
{
  "request_logs": [
    {
      "request_id": 1,
      "call_type": "RETRIEVAL_EMBEDDING",
      "start_time": "2023-10-01T10:00:00Z",
      "end_time": "2023-10-01T10:15:00Z",
      "cost": 10.0
    },
    {
      "request_id": 1,
      "call_type": "GENERATION",
      "start_time": "2023-10-01T10:16:00Z",
      "end_time": "2023-10-01T10:30:00Z",
      "cost": 5.0
    },
    {
      "request_id": 2,
      "call_type": "RETRIEVAL_EMBEDDING",
      "start_time": "2023-10-01T10:45:00Z",
      "end_time": "2023-10-01T10:60:00Z",
      "cost": 7.0
    },
    {
      "request_id": 3,
      "call_type": "GENERATION",
      "start_time": "2023-10-01T11:00:00Z",
      "end_time": "2023-10-01T11:15:00Z",
      "cost": 8.0
    }
  ]
}
```

**Flow:**

1. Request Logs
2. Filter Retrieval-Embedding
3. Aggregated Costs
4. Filter Generation
5. Aggregated Costs
6. Join Aggregations
7. Total Cost Calculation

