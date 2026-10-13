---
type: "practice"
difficulty: "Expert"
---

# How would you design a SQL rollup that maintains daily feature aggregates incrementally so retraining reads pre-summarized rows instead of re-scanning raw events, and how do you handle corrections?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Incremental Rollups](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Incremental_Rollups.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design an efficient SQL rollup for maintaining daily feature aggregates incrementally in a machine learning (ML) context, we need to ensure that retraining processes can leverage pre-summarized rows instead of re-scanning raw events. This approach minimizes the computational overhead and optimizes resource usage. The key challenge is handling corrections efficiently while ensuring data consistency.

The core idea is to create an incremental rollup table where daily feature aggregates are stored incrementally, allowing for efficient querying without reprocessing large volumes of raw data. Corrections can be handled by updating specific rows in the rollup table rather than recalculating entire segments.

#### Triage
Question: How do you design a SQL rollup to maintain daily feature aggregates incrementally, ensuring that retraining processes read pre-summarized rows instead of re-scanning raw events?
What is being probed: Design principles for efficient data summarization and handling corrections in SQL.

#### Assumptions
The system needs to handle large volumes of raw event data efficiently.
Corrections might be required, necessitating updates to the rollup table.

#### First Principles Solution

Table Structure:

Design a schema that captures daily feature aggregates with appropriate columns for time granularity and relevant features.

Incremental Aggregation:

Implement a strategy where new data is inserted into the raw event table, and corresponding entries are updated in the rollup table.

Handling Corrections:

Develop procedures to update specific rows in the rollup table when corrections are needed without recalculating entire segments.
Detailed Implementation

Schema Design

SQL
CREATE TABLE RawEvents (
    event_id INT PRIMARY KEY,
    feature_name VARCHAR(255),
    value DECIMAL(10, 4),
    timestamp TIMESTAMP
);

CREATE TABLE FeatureRollup (
    feature_name VARCHAR(255) NOT NULL,
    date DATE NOT NULL,
    daily_sum DECIMAL(18, 6),
    PRIMARY KEY (feature_name, date)
);

Incremental Aggregation

During the ingestion process, raw events are inserted into RawEvents.
A scheduled job or trigger updates the rollup table by aggregating new values.

Handling Corrections

When a correction is required, find the specific entry in the rollup table and update it directly.
SQL
UPDATE FeatureRollup
SET daily_sum = (SELECT SUM(value) FROM RawEvents WHERE feature_name = 'example_feature' AND date = '2023-10-05')
WHERE feature_name = 'example_feature' AND date = '2023-10-05';

#### Common Mistakes
and Follow-Ups
Mistake 1: Failing to index the rollup table properly, leading to slower query performance.
Mistake 2: Ignoring the need for robust error handling during corrections.

#### Follow-Up
Discussing how to handle edge cases such as multiple simultaneous corrections or large-scale retraining processes.

#### Closing Summary

In summary, designing an efficient SQL rollup involves carefully structuring the database schema, implementing incremental aggregation strategies, and developing effective procedures for handling corrections. This approach ensures that retraining processes can leverage pre-summarized rows efficiently while maintaining data integrity.

**Code:**

```sql
-- Schema Design
CREATE TABLE RawEvents (
    event_id INT PRIMARY KEY,
    feature_name VARCHAR(255),
    value DECIMAL(10, 4),
    timestamp TIMESTAMP
);

CREATE TABLE FeatureRollup (
    feature_name VARCHAR(255) NOT NULL,
    date DATE NOT NULL,
    daily_sum DECIMAL(18, 6),
    PRIMARY KEY (feature_name, date)
);
```

