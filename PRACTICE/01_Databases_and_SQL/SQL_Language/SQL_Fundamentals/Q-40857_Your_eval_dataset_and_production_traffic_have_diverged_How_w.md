---
type: "practice"
difficulty: "Deep"
---

# Your eval dataset and production traffic have diverged. How would you write SQL comparing the distribution of query intents in your eval table versus a sampled production-log table?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Analytical SQL for ML Features](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Analytical_SQL_for_ML_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When evaluating an AI system, it's crucial to ensure that the model performs well across both evaluation and production environments. Given that the eval dataset and production traffic have diverged, writing SQL to compare the distribution of query intents between these two datasets is essential for identifying any discrepancies or biases. This involves extracting a representative sample from the production logs and comparing it with the eval data.

#### Triage

The question asks us to write SQL to compare the distribution of query intents in an evaluation (eval) dataset against a sampled production log table. We need to ensure that our sampling process is robust enough to capture the true distribution of queries seen in production, as differences can indicate potential issues with model performance or data skew.

#### First-Principles Solution
Define Query Intents: Identify the query intent columns or features in both tables.
Sampling Production Data: Extract a representative sample from the production log table to ensure it reflects the true distribution of queries.
Aggregation and Comparison: Use SQL aggregation functions to compare the distributions.
Step-by-Step Solution
1. Define Query Intents

Let's assume our query intents are captured in columns query_type and priority_level.

SQL
-- Example schema for eval table
CREATE TABLE eval_queries (
    id INT,
    user_id INT,
    query_type VARCHAR(255),
    priority_level INT
);

-- Example schema for production logs
CREATE TABLE prod_logs (
    log_id INT PRIMARY KEY,
    user_id INT,
    query_type VARCHAR(255),
    priority_level INT,
    timestamp TIMESTAMP
);
2. Sampling Production Data

We need to select a representative sample of the production logs. A common approach is to use ROW_NUMBER() and LIMIT to ensure randomness.

SQL
WITH sampled_logs AS (
    SELECT *
    FROM prod_logs
    WHERE (user_id, timestamp) IN (
        SELECT user_id, timestamp
        FROM prod_logs
        ORDER BY RAND()
        LIMIT 10000 -- Adjust the sample size as needed
    )
)
3. Aggregation and Comparison
SQL
-- Step 1: Count queries by intent in eval table
SELECT
    query_type,
    priority_level,
    COUNT(*) AS eval_count
FROM eval_queries
GROUP BY query_type, priority_level;

-- Step 2: Count queries by intent in sampled production logs
SELECT
    query_type,
    priority_level,
    COUNT(*) AS prod_count
FROM sampled_logs
GROUP BY query_type, priority_level;

-- Step 3: Join and compare counts
WITH eval_counts AS (
    SELECT
        query_type,
        priority_level,
        COUNT(*) AS eval_count
    FROM eval_queries
    GROUP BY query_type, priority_level
),
prod_counts AS (
    SELECT
        query_type,
        priority_level,
        COUNT(*) AS prod_count
    FROM sampled_logs
    GROUP BY query_type, priority_level
)
SELECT
    ec.query_type,
    ec.priority_level,
    ec.eval_count,
    pc.prod_count,
    (pc.prod_count - ec.eval_count) / NULLIF(ec.eval_count, 0) * 100 AS diff_percent
FROM eval_counts ec
LEFT JOIN prod_counts pc ON ec.query_type = pc.query_type AND ec.priority_level = pc.priority_level;

#### Closing Summary
and Common Mistakes/Follow-Ups

The SQL solution above provides a comprehensive comparison of query intents between the evaluation dataset and sampled production logs. It ensures that we account for both positive (presence) and negative (absence) cases by calculating percentage differences.

#### Common Mistakes
Sampling Bias: Ensure random sampling is used to avoid skewing results.
Sample Size: The sample size should be large enough to represent the true distribution of production queries but small enough to manage.
Normalization: Use percentages or ratios to normalize the comparison across different query types and priority levels.

#### Follow-Ups
Adjust Sample Size: Depending on the volume of data, you might need to adjust the sample size.
Additional Metrics: Consider adding additional metrics such as average response time for each intent type.
Temporal Aggregation: If needed, aggregate results over different time periods (e.g., hourly, daily) to get a more granular view.

**Comparison:**

| query_type | priority_level | eval_count | prod_count | diff_percent |
|------------|----------------|------------|------------|--------------|
| search     | 1              | 500        | 498        | -0.4%        |
| recommend  | 2              | 300        | 305        | +1.6%        |

**Code:**

```sql
-- Example SQL queries
WITH sampled_logs AS (
    SELECT *
    FROM prod_logs
    WHERE (user_id, timestamp) IN (
        SELECT user_id, timestamp
        FROM prod_logs
        ORDER BY RAND()
        LIMIT 10000 -- Adjust the sample size as needed
    )
),
eval_counts AS (
    SELECT
        query_type,
        priority_level,
        COUNT(*) AS eval_count
    FROM eval_queries
    GROUP BY query_type, priority_level
),
prod_counts AS (
    SELECT
        query_type,
        priority_level,
        COUNT(*) AS prod_count
    FROM sampled_logs
    GROUP BY query_type, priority_level
)
SELECT
    ec.query_type,
    ec.priority_level,
    ec.eval_count,
    pc.prod_count,
    (pc.prod_count - ec.eval_count) / NULLIF(ec.eval_count, 0) * 100 AS diff_percent
FROM eval_counts ec
LEFT JOIN prod_counts pc ON ec.query_type = pc.query_type AND ec.priority_level = pc.priority_level;
```

**Example input / output:**

```text
-- Input: eval_queries and prod_logs tables as defined above

-- Output:
-- query_type | priority_level | eval_count | prod_count | diff_percent
-- ----------------------------------------------
-- 'search'   | 1             | 500        | 498        | -0.4%
-- 'recommend'| 2             | 300        | 305        | +1.6%
```

**Data:**

```sql
-- Example schema for eval table
CREATE TABLE eval_queries (
    id INT,
    user_id INT,
    query_type VARCHAR(255),
    priority_level INT
);

-- Example schema for production logs
CREATE TABLE prod_logs (
    log_id INT PRIMARY KEY,
    user_id INT,
    query_type VARCHAR(255),
    priority_level INT,
    timestamp TIMESTAMP
);
```

**Flow:**

1. Define Query Intents
2. Sample Production Logs
3. Aggregate Eval Data
4. Aggregate Sampled Prod Data
5. Join and Compare Counts
6. Output Results
7. Analyze Differences
8. Adjust Sample Size ifNecessary
9. Additional Metrics?
10. Temporal Aggregation?
11. Final Output

