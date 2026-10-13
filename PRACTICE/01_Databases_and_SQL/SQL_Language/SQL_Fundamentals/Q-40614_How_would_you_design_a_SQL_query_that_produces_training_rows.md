---
type: "practice"
difficulty: "Expert"
---

# How would you design a SQL query that produces training rows only from features that were observable at each historical prediction cutoff, using a per-row asof timestamp column?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Analytical SQL for ML Features](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Analytical_SQL_for_ML_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, we need to design an SQL query that filters out training rows based on whether certain features are observable at specific historical prediction cutoffs. This involves using an asof join or a similar mechanism to match the current row with its relevant historical data.

#### Triaging the Question

The core of the problem is understanding the concept of "historical prediction cutoffs" and ensuring that only features available up to that point are used for training. The interviewer might be probing your knowledge of SQL joins, temporal data handling, and conditional filtering mechanisms. We will assume a table structure where each row has an asof timestamp indicating the latest known state before the prediction.

#### First-Principles Solution

We start by defining our tables:

Features Table: Contains historical feature values up to a certain cutoff.
Predictions Table: Has rows for predictions at different timestamps, with an asof column indicating the historical timestamp cutoff.

The query will perform an asof join between these two tables based on the asof timestamp from the Predictions table and ensure that only features up to this timestamp are considered. Here’s a step-by-step approach:

Join Condition: Match each prediction row with its corresponding feature values using the asof timestamp.
Filtering: Ensure that for each prediction, all features used are strictly before or at the cutoff.
Detailed Steps

Let's assume we have two tables:

features: Contains historical feature data with columns id, feature_name, and value.
predictions: Has predictions at different timestamps with columns prediction_id, asof_timestamp, and other relevant features like prediction_value.

The SQL query would look something like this:

SQL
SELECT
    p.prediction_id,
    f.feature_name,
    f.value AS feature_value,
    p.asof_timestamp,
    p.prediction_value
FROM
    predictions p
JOIN
    (SELECT
         id,
         feature_name,
         value,
         asof_timestamp
     FROM
         features
     WHERE
         asof_timestamp <= p.asof_timestamp) f
ON
    p.asof_timestamp = f.asof_timestamp;

This query ensures that only the relevant historical data is used for each prediction.

#### Closing Summary
and Common Mistakes/Follow-Ups

The key takeaway is understanding how to use asof joins or similar mechanisms in SQL to handle temporal data effectively. A common mistake might be incorrectly joining on timestamps without filtering properly, leading to data leakage where future information influences predictions.

For follow-ups, you could discuss:

#### Performance considerations
for large datasets.
Handling missing values or gaps in the feature dataset.
Using window functions if asof joins are not available in your SQL dialect.

**Comparison:**

```
Features Table
asof_timestamp
Predictions Table
asof_timestamp
Join Condition
asof_timestamp =asof_timestamp
Result Table
feature_value, prediction_value
```

**Code:**

```sql
SELECT
    p.prediction_id,
    f.feature_name,
    f.value AS feature_value,
    p.asof_timestamp,
    p.prediction_value
FROM
    predictions p
JOIN
    (SELECT
         id,
         feature_name,
         value,
         asof_timestamp
     FROM
         features
     WHERE
         asof_timestamp <= p.asof_timestamp) f
ON
    p.asof_timestamp = f.asof_timestamp;
```

**Example input / output:**

```

INSERT INTO features (id, feature_name, value, asof_timestamp) VALUES (1, 'feature_A', 5.5, '2023-01-01'), (2, 'feature_B', 7.8, '2023-02-01'), (3, 'feature_C', 9.0, '2023-03-01');

INSERT INTO predictions (prediction_id, asof_timestamp, prediction_value) VALUES (1, '2023-01-15', 6.0), (2, '2023-02-28', 7.9), (3, '2023-03-31', 8.5);

CODE
Copy

```sql
SELECT
    p.prediction_id,
    f.feature_name,
    f.value AS feature_value,
    p.asof_timestamp,
    p.prediction_value
FROM
    predictions p
JOIN
    (SELECT
         id,
         feature_name,
         value,
         asof_timestamp
     FROM
         features
     WHERE
         asof_timestamp <= p.asof_timestamp) f
ON
    p.asof_timestamp = f.asof_timestamp;

Result:

SQL
Copy
+----------------+------------+-------------+---------------------+------------------+
| prediction_id   | feature_name| feature_value| asof_timestamp      | prediction_value |
+----------------+------------+-------------+---------------------+------------------+
| 1              | feature_A   | 5.5         | 2023-01-01T00:00:00 | 6.0             |
| 2              | feature_B   | 7.8         | 2023-02-01T00:00:00 | 7.9             |
| 3              | feature_C   | NULL        | 2023-03-31T00:00:00 | 8.5             |
+----------------+------------+-------------+---------------------+------------------+
```

**Data:**

```sql
CREATE TABLE features (
    id INT,
    feature_name VARCHAR(50),
    value DECIMAL(10, 2),
    asof_timestamp TIMESTAMP
);

CREATE TABLE predictions (
    prediction_id INT,
    asof_timestamp TIMESTAMP,
    prediction_value DECIMAL(10, 2)
);
```

**Flow:**

1. Features Table
2. asof_timestamp
3. Predictions Table
4. asof_timestamp
5. Join Condition
6. asof_timestamp =asof_timestamp
7. Result Table
8. feature_value, prediction_value

