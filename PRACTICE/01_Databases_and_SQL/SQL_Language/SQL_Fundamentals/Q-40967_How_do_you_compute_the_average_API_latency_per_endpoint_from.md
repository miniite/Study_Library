---
type: "practice"
difficulty: "Intermediate"
---

# How do you compute the average API latency per endpoint from a request-log table, and why exclude NULL latencies?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Data Completeness Metrics](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Data_Completeness_Metrics.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute the average API latency per endpoint from a request-log table, we need to extract the relevant fields and handle edge cases like NULL values. The key is ensuring that only valid latency data contributes to our calculation.

#### Triaging the Question

The core of this problem involves filtering out invalid or missing latencies (NULLs) before calculating the average latency per endpoint. This requires a basic understanding of SQL, particularly how to use aggregate functions and handle nulls properly.

#### First-Principles Solution
Identify Relevant Columns: We need columns for endpoint and latency.
Filter NULL Latencies: Exclude rows where the latency is NULL.
Group By Endpoint: Aggregate latencies by endpoint to calculate the average.
Handle Edge Cases: Ensure that if there are no non-null latencies, we return a meaningful result (e.g., 0 or NULL).
SQL Query

Here's the step-by-step SQL query:

SQL
SELECT
    endpoint,
    AVG(latency) AS avg_latency
FROM request_logs
WHERE latency IS NOT NULL
GROUP BY endpoint;
Explanation
SELECT endpoint, AVG(latency) AS avg_latency: We select the endpoint column and calculate the average of non-null latencies.
FROM request_logs: The table from which we are retrieving data.
WHERE latency IS NOT NULL: Filter out rows where latency is NULL to avoid affecting the average calculation.
GROUP BY endpoint: Group results by each unique endpoint, ensuring that the average is calculated per endpoint.

#### Closing Summary
and Common Mistakes

This query ensures accurate computation of average latencies by excluding NULL values, providing a more reliable metric. A common mistake might be forgetting to filter out NULLs, leading to incorrect averages.

#### Example
Input-Output

Suppose we have a request_logs table with the following data:

id	endpoint	latency
1	/api/v1/auth	500
2	/api/v2/user	NULL
3	/api/v1/auth	600
4	/api/v2/user	700

The expected output would be:

endpoint	avg_latency
/api/v1/auth	550
/api/v2/user	700
Self-Check

Correctness and completeness confirmed.

**Comparison:**

| id | endpoint    | latency |
|----|-------------|---------|
| 1  | /api/v1/auth | 500     |
| 2  | /api/v2/user | NULL    |
| 3  | /api/v1/auth | 600     |
| 4  | /api/v2/user | 700     |
| endpoint    | avg_latency |
|-------------|-------------|
| /api/v1/auth | 550         |
| /api/v2/user | 700         |

**Code:**

```sql
SELECT
    endpoint,
    AVG(latency) AS avg_latency
FROM request_logs
WHERE latency IS NOT NULL
GROUP BY endpoint;
```

**Example input / output:**

```text
| id | endpoint    | latency |
|----|-------------|---------|
| 1  | /api/v1/auth | 500     |
| 2  | /api/v2/user | NULL    |
| 3  | /api/v1/auth | 600     |
| 4  | /api/v2/user | 700     |

Output:

| endpoint    | avg_latency |
|-------------|-------------|
| /api/v1/auth | 550         |
| /api/v2/user | 700         |
```

**Data:**

```json
[
  {"id": 1, "endpoint": "/api/v1/auth", "latency": 500},
  {"id": 2, "endpoint": "/api/v2/user", "latency": null},
  {"id": 3, "endpoint": "/api/v1/auth", "latency": 600},
  {"id": 4, "endpoint": "/api/v2/user", "latency": 700}
]
```

**Flow:**

1. Yes
2. No
3. Start
4. Is latency NULL?
5. Skip row
6. Calculate AVG(latency) perendpoint
7. Output results
8. Group by endpoint and outputresults
9. End
10. End

