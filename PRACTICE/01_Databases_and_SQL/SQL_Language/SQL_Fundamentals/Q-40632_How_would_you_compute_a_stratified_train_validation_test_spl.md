---
type: "practice"
difficulty: "Deep"
---

# How would you compute a stratified train/validation/test split in SQL that preserves the label ratio within each entity segment and is stable across re-runs via a hash of the key?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Analytical SQL for ML Features](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Analytical_SQL_for_ML_Features.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the task of computing a stratified train/validation/test split in SQL, we need to ensure that the distribution of labels remains consistent within each entity segment while maintaining stability across re-runs using a hash of the key. This involves partitioning the dataset into segments based on keys (like user IDs or product IDs), then splitting each segment proportionally according to predefined ratios for training, validation, and testing.

The core idea is to use a deterministic method that consistently assigns data points to the same split across multiple executions. This can be achieved by hashing the key and using the hash value to determine which split each record belongs to.

First, let's define the problem more clearly:

We need to stratify the data based on labels within each segment defined by keys.
The splits must be reproducible: a consistent seed or deterministic method should ensure that the same records are assigned to the same splits over multiple executions.

Now, we'll go through this step-by-step from first principles.

Define Segments and Ratios: Assume we have a table data with columns id, label, and any other relevant features.
Hashing Key for Stability: Use a hash function to generate a deterministic value based on the key, ensuring that the same keys always produce the same values.
Stratified Sampling within Segments: Ensure that each segment's label distribution is preserved in the split.

Here’s how you can implement this:

SQL
WITH segments AS (
    SELECT
        id,
        label,
        -- Use a hash function to ensure deterministic splitting
        MD5(id) AS segment_hash
    FROM data
),
ratios AS (
    -- Define your desired split ratios (e.g., 60% train, 20% validation, 20% test)
    SELECT
        'train' AS split,
        0.60 as ratio
    UNION ALL
    SELECT
        'validation' AS split,
        0.20 as ratio
    UNION ALL
    SELECT
        'test' AS split,
        0.20 as ratio
),
assigned_splits AS (
    SELECT
        s.id,
        s.label,
        r.split,
        -- Determine the split based on the hash value and predefined ratios
        CASE
            WHEN RAND() * 100 <= (s.segment_hash % 100) * 100 * r.ratio THEN r.split
            ELSE 'test'
        END AS final_split
    FROM segments s
    CROSS JOIN ratios r
)
SELECT
    id,
    label,
    final_split
FROM assigned_splits
ORDER BY id;

#### Closing Summary
and Common Mistakes/Follow-Ups

This SQL query uses a combination of hashing and randomization to ensure that the splits are reproducible and stratified. The MD5 function provides a deterministic hash value based on the key, and the RAND() function is used for random assignment within each segment.

#### Common Mistakes

Incorrect Randomization: Using RAND() alone can introduce variability across re-runs.
Insufficient Segment Definition: Not partitioning by keys (e.g., user IDs) may lead to incorrect stratification if the labels are not evenly distributed.
Hash Function Choice: While MD5 is deterministic, it might be worth exploring other hash functions or techniques for better distribution.

#### Follow-Ups

What happens if the dataset changes?
How would you optimize this query for larger datasets?
Can you implement this in a different database system?

**Comparison:**

```
segments
Compute Hash
ratios
Assign Splits
Final Split Selection
Output Data
```

**Code:**

```sql
WITH segments AS (
    SELECT
        id,
        label,
        MD5(id) AS segment_hash
    FROM data
),
ratios AS (
    SELECT
        'train' AS split,
        0.60 as ratio
    UNION ALL
    SELECT
        'validation' AS split,
        0.20 as ratio
    UNION ALL
    SELECT
        'test' AS split,
        0.20 as ratio
),
assigned_splits AS (
    SELECT
        s.id,
        s.label,
        r.split,
        CASE
            WHEN RAND() * 100 <= (s.segment_hash % 100) * 100 * r.ratio THEN r.split
            ELSE 'test'
        END AS final_split
    FROM segments s
    CROSS JOIN ratios r
)
SELECT
    id,
    label,
    final_split
FROM assigned_splits
ORDER BY id;
```

**Example input / output:**

```text
WITH segments AS (
    SELECT
        1 AS id,
        'A' AS label
    UNION ALL
    SELECT
        2,
        'B'
    UNION ALL
    SELECT
        3,
        'C'
),
ratios AS (
    SELECT
        'train' AS split,
        0.60 as ratio
    UNION ALL
    SELECT
        'validation' AS split,
        0.20 as ratio
    UNION ALL
    SELECT
        'test' AS split,
        0.20 as ratio
),
assigned_splits AS (
    SELECT
        s.id,
        s.label,
        r.split,
        CASE
            WHEN RAND() * 100 <= (s.id % 100) * 100 * r.ratio THEN r.split
            ELSE 'test'
        END AS final_split
    FROM segments s
    CROSS JOIN ratios r
)
SELECT
    id,
    label,
    final_split
FROM assigned_splits
ORDER BY id;
```

**Data:**

```sql
-- Sample data table structure for illustration
CREATE TABLE data (
    id INT PRIMARY KEY,
    label VARCHAR(10),
    feature1 FLOAT,
    feature2 FLOAT
);
```

