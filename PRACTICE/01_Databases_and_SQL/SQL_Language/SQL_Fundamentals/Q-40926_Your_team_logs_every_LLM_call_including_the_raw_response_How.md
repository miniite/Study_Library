---
type: "practice"
difficulty: "Deep"
---

# Your team logs every LLM call including the raw response. How would you enforce a retention policy in SQL that anonymizes prompts older than 30 days without dropping aggregate metrics?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Idempotency Key Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Idempotency_Key_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To enforce a retention policy that anonymizes prompts older than 30 days while preserving aggregate metrics, we need to carefully balance data privacy with operational efficiency. The key steps involve creating a staging table for anonymization, implementing an ETL process using SQL, and ensuring the integrity of our analytics.

#### Triage

The question is essentially asking how to design an SQL-based system that can automatically cleanse sensitive data (prompts) older than 30 days while still maintaining the ability to aggregate necessary metrics. The core challenge here involves handling large volumes of data efficiently and ensuring that the anonymization process does not interfere with analytics.

#### First-Principles Solution
Create a Staging Table: Create a new table to store the anonymized data.
ETL Process: Use SQL to perform an ETL (Extract, Transform, Load) operation where sensitive data is anonymized and moved from the original table to the staging table.
Update Aggregates: After anonymizing the prompts, update the aggregate metrics to reflect changes in the raw data.
Step-by-Step Solution
1. Create a Staging Table
SQL
CREATE TABLE logs_anonymized (
    id INT PRIMARY KEY,
    timestamp TIMESTAMP NOT NULL,
    model_version VARCHAR(255),
    prompt TEXT,
    response TEXT,
    -- other columns as needed
);
2. ETL Process to Anonymize Data

We will use a combination of SQL functions and conditional statements to anonymize the prompts older than 30 days.

SQL
-- Step 1: Create an intermediate table with only the fields we need to keep for aggregation
CREATE TABLE logs_to_analyze AS
SELECT id, timestamp, model_version, prompt, response
FROM log_calls
WHERE timestamp < CURRENT_DATE - INTERVAL '30' DAY;

-- Step 2: Anonymize prompts in the intermediate table
UPDATE logs_to_analyze SET prompt = 'ANONYMIZED';

-- Step 3: Move anonymized data to the staging table
INSERT INTO logs_anonymized (id, timestamp, model_version, prompt, response)
SELECT id, timestamp, model_version, prompt, response
FROM logs_to_analyze;

-- Step 4: Drop the intermediate table if no further use is needed
DROP TABLE IF EXISTS logs_to_analyze;
3. Update Aggregate Metrics

To ensure that our aggregate metrics are updated correctly after anonymization:

SQL
-- Example of updating a simple count metric
UPDATE metrics
SET prompt_count = (SELECT COUNT(*) FROM log_calls WHERE timestamp >= CURRENT_DATE - INTERVAL '30' DAY)
WHERE name = 'prompt_count';

#### Closing Summary
and Common Mistakes

This approach ensures that sensitive data is anonymized while maintaining the integrity of our analytics. The key points to remember are:

Staging Table: Use a staging table for intermediate storage.
ETL Process: Implement an ETL process using SQL functions to anonymize data efficiently.
Aggregates Update: Ensure aggregate metrics reflect changes in the raw data accurately.

#### Common Mistakes
Forgetting to use a Staging Table: Directly modifying the main log table can lead to data loss or inconsistencies.
Not Handling Aggregates Correctly: Failing to update aggregations can result in inaccurate reporting.
Performance Issues: Not optimizing the ETL process can lead to performance bottlenecks, especially with large datasets.

**Comparison:**

| id | timestamp        | model_version | prompt                  | response     |
|----|------------------|---------------|-------------------------|--------------|
| 1  | 2023-05-01       | v1.0          | Example prompt          | Response text|
| 2  | 2023-04-30       | v1.1          | ANONYMIZED              | Different response|
| name        | value |
|-------------|-------|
| prompt_count | 150   |

**Code:**

```

CREATE TABLE logs_anonymized ( id INT PRIMARY KEY, timestamp TIMESTAMP NOT NULL, model_version VARCHAR(255), prompt TEXT, response TEXT );

CREATE TABLE logs_to_analyze AS SELECT id, timestamp, model_version, prompt, response FROM log_calls WHERE timestamp < CURRENT_DATE - INTERVAL '30' DAY;

UPDATE logs_to_analyze SET prompt = 'ANONYMIZED';

INSERT INTO logs_anonymized (id, timestamp, model_version, prompt, response) SELECT id, timestamp, model_version, prompt, response FROM logs_to_analyze;

DROP TABLE IF EXISTS logs_to_analyze;

CODE
Copy

```sql
UPDATE metrics
SET prompt_count = (SELECT COUNT(*) FROM log_calls WHERE timestamp >= CURRENT_DATE - INTERVAL '30' DAY)
WHERE name = 'prompt_count';
```

**Example input / output:**

```text
-- Input - Data in log_calls table before anonymization

INSERT INTO logs_anonymized (id, timestamp, model_version, prompt, response)
SELECT id, timestamp, model_version, prompt, response
FROM (
  SELECT
    id,
    timestamp,
    model_version,
    CASE WHEN timestamp < CURRENT_DATE - INTERVAL '30' DAY THEN 'ANONYMIZED' ELSE prompt END AS prompt,
    response
  FROM log_calls
) subquery;

-- Output - Anonymized data in logs_anonymized table and updated metrics

SELECT COUNT(*) FROM log_calls WHERE timestamp >= CURRENT_DATE - INTERVAL '30' DAY; -- New count after anonymization
```

**Data:**

```json
{
  "log_calls": [
    {
      "id": 1,
      "timestamp": "2023-05-01",
      "model_version": "v1.0",
      "prompt": "Example prompt",
      "response": "Response text"
    },
    {
      "id": 2,
      "timestamp": "2023-04-30",
      "model_version": "v1.1",
      "prompt": "Another example",
      "response": "Different response"
    }
  ],
  "metrics": [
    {
      "name": "prompt_count",
      "value": 150
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Update metric
4. ETL Process
5. Is timestamp < 30 days old?
6. Anonymize prompt
7. Keep original data
8. Move to logs_anonymizedtable
9. Skip anonymization
10. Maintain metrics integrity
11. metrics table updated

