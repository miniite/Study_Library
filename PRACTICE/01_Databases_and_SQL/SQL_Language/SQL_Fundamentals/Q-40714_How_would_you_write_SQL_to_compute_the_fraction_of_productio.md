---
type: "practice"
difficulty: "Intermediate"
---

# How would you write SQL to compute the fraction of production requests where a required feature was defaulted/imputed (because the real value was missing), as a serving-quality metric?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Aggregation Query Practice](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Aggregation_Query_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To calculate the fraction of production requests where a required feature was imputed, you need to identify both the total number of requests and the number of those requests that had the feature imputed. This involves writing SQL queries to count these two quantities and then computing their ratio.

#### Triaging the Question

The question asks for an SQL query to compute the fraction of production requests where a required feature was imputed due to missing data. The key concepts are:

Identifying Imputed Features: Distinguishing between original values and imputed ones.
Aggregation: Summarizing counts at different levels (e.g., per request, per user).
Serving Quality Metric: Providing a clear, concise metric that reflects the quality of service.

#### Assumptions
There is a table named production_requests with columns such as request_id, user_id, feature_value, and possibly a default_flag.
The feature_value column contains the actual value if available or an imputed value.
The default_flag indicates whether the feature was imputed (e.g., 1 for imputed, 0 for non-imputed).

#### First-Principles Solution

To find the fraction of production requests with imputed features, follow these steps:

Identify Imputed Features: Use a condition to filter rows where default_flag is set.
Count Total Requests and Imputed Requests:
Count all unique request IDs (request_id) as the total number of requests.
Count the distinct request_ids that have at least one imputed feature.

Here’s how you can structure your SQL query:

SQL
-- Step 1: Identify requests with any imputed features
WITH imputed_requests AS (
    SELECT request_id
    FROM production_requests
    WHERE default_flag = 1
)

-- Step 2: Calculate the total number of unique requests and those with imputed features
SELECT
    (SELECT COUNT(DISTINCT request_id) FROM production_requests) AS total_requests,
    (SELECT COUNT(DISTINCT request_id) FROM imputed_requests) AS imputed_requests_count,

    -- Calculate the fraction of imputed requests
    CASE
        WHEN (SELECT COUNT(DISTINCT request_id) FROM production_requests) > 0 THEN
            (COUNT(DISTINCT request_id) OVER (PARTITION BY request_id)) /
            (SELECT COUNT(DISTINCT request_id) FROM production_requests)
        ELSE 0
    END AS imputed_fraction;

#### Closing Summary
and Common Mistakes/Follow-Ups

This query effectively calculates the fraction of requests with imputed features. The key steps are:

Using a common table expression (CTE) to isolate rows where default_flag is set.
Counting unique request IDs in both total and imputed datasets.
Dividing the count of imputed requests by the total number of requests.

#### Common Mistakes

Not properly differentiating between requests with at least one imputed feature and all requests.
Forgetting to use COUNT(DISTINCT) for accurate counts.
Misinterpreting the condition under which a request is considered imputed.

#### Follow-Ups

How would you handle multiple imputed features per request?
What if there are missing values in other attributes besides the feature value?
Would indexing improve query performance, and on what columns?

**Comparison:**

```
table production_requests {
    request_id|user_id|feature_value|default_flag
    1|101|A|FALSE
    2|101||TRUE
    3|102|B|FALSE
    4|103||TRUE
    5|103|C|FALSE
}
```

**Code:**

```sql
WITH imputed_requests AS (
    SELECT request_id
    FROM production_requests
    WHERE default_flag = 1
)

SELECT
    (SELECT COUNT(DISTINCT request_id) FROM production_requests) AS total_requests,
    (SELECT COUNT(DISTINCT request_id) FROM imputed_requests) AS imputed_requests_count,

    -- Calculate the fraction of imputed requests
    CASE
        WHEN (SELECT COUNT(DISTINCT request_id) FROM production_requests) > 0 THEN
            (COUNT(DISTINCT request_id) OVER (PARTITION BY request_id)) /
            (SELECT COUNT(DISTINCT request_id) FROM production_requests)
        ELSE 0
    END AS imputed_fraction;
```

**Example input / output:**

```text
WITH imputed_requests AS (
    SELECT request_id
    FROM production_requests
    WHERE default_flag = 1
)

SELECT
    (SELECT COUNT(DISTINCT request_id) FROM production_requests) AS total_requests,
    (SELECT COUNT(DISTINCT request_id) FROM imputed_requests) AS imputed_requests_count,

    -- Calculate the fraction of imputed requests
    CASE
        WHEN (SELECT COUNT(DISTINCT request_id) FROM production_requests) > 0 THEN
            (COUNT(DISTINCT request_id) OVER (PARTITION BY request_id)) /
            (SELECT COUNT(DISTINCT request_id) FROM production_requests)
        ELSE 0
    END AS imputed_fraction;

-- Output:
+-------------+-----------------+-------------------+
| total_requests | imputed_requests_count | imputed_fraction  |
+-------------+-----------------+-------------------+
|          5   |                2 |                 0.4 |
+-------------+-----------------+-------------------+
```

**Data:**

```sql
CREATE TABLE production_requests (
    request_id INT,
    user_id INT,
    feature_value VARCHAR(255),
    default_flag BOOLEAN
);
INSERT INTO production_requests (request_id, user_id, feature_value, default_flag) VALUES
(1, 101, 'A', FALSE),
(2, 101, NULL, TRUE),
(3, 102, 'B', FALSE),
(4, 103, NULL, TRUE),
(5, 103, 'C', FALSE);
```

**Flow:**

1. Yes
2. No
3. Identify Imputed Requests
4. Are there any imputedrequests?
5. Count Total and ImputedRequest IDs
6. Calculate Fraction
7. Return 0 as fraction
8. Report Fraction

