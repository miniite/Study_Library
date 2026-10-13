---
type: "practice"
difficulty: "Basic"
---

# How would you count NULLs per column in a metrics feature table to assess missing telemetry before modeling?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ NULL Handling](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/NULL_Handling.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To count NULLs per column in a metrics feature table, the key is to use SQL aggregate functions and ensure data integrity before feeding it into machine learning models. This process involves querying each column to identify missing values (NULLs) and understanding their distribution across different columns.

First, let's understand what we are probing: the core of this question lies in writing a SQL query that can efficiently count NULLs for every column in a given metrics feature table. The interviewer is also likely looking at how you handle null values, which can significantly impact data quality and model performance.

Assuming we have a table called metrics_feature with several columns like cpu_usage, memory_usage, etc., our goal is to identify the number of NULLs in each column. Here’s a step-by-step breakdown:

Identify Columns: List all columns in the metrics feature table.
Write SQL Query: Use a combination of SQL functions such as COUNT and CASE WHEN to count NULL values per column.

Now, let's walk through the first-principles solution:

#### First-Principles Solution

First, we need to identify the columns in the metrics_feature table. Let’s assume it has 5 columns: cpu_usage, memory_usage, network_in, network_out, and disk_usage.

SQL
-- List of columns for which NULL values are being counted
SELECT
    column_name,
    COUNT(CASE WHEN column_name IS NULL THEN 1 ELSE NULL END) AS null_count
FROM (
    SELECT 'cpu_usage' as column_name UNION ALL
    SELECT 'memory_usage' UNION ALL
    SELECT 'network_in' UNION ALL
    SELECT 'network_out' UNION ALL
    SELECT 'disk_usage'
) x
LEFT JOIN metrics_feature mf ON 1=0
GROUP BY column_name;
Explanation
Step 1: We create a subquery that lists all the columns in the metrics_feature table.
Step 2: For each column, we use a CASE WHEN statement to count NULL values. This is done by checking if the value is NULL and incrementing a counter when it is.
Step 3: We perform a LEFT JOIN with an empty table (using ON 1=0) to ensure that every column in the subquery gets counted, even if there are no corresponding rows in the metrics_feature table.
Step 4: Finally, we group by each column and count the NULLs.

#### Example
Output
column_name	null_count
cpu_usage	32
memory_usage	19
network_in	58
network_out	47
disk_usage	0

This output helps us understand where the missing data (NULLs) are more prevalent, which can be crucial for deciding how to handle them in our data preprocessing pipeline.

#### Closing Summary

By writing a SQL query that counts NULL values per column, we ensure that any gaps or inconsistencies in our telemetry data are identified early on. This is essential before modeling, as missing values can lead to biased or inaccurate models.

#### Common Mistakes
and Follow-Ups
Mistake 1: Forgetting to handle columns dynamically if the table schema changes.

#### Follow-Up
1: Discuss dynamic SQL or using a script that lists columns automatically rather than hardcoding them.
Mistake 2: Not considering NULLs in your data quality checks and preprocessing steps.

#### Follow-Up
2: Explain how to integrate this step into a broader ETL process, ensuring data quality is maintained throughout.

**Comparison:**

|column_name | null_count|
|------------|-----------|
|cpu_usage   | 1         |
|memory_usage| 1         |
|network_in  | 0         |
|network_out | 0         |
|disk_usage  | 1         |

**Code:**

```sql
-- SQL Query for counting NULL values per column
SELECT
    column_name,
    COUNT(CASE WHEN column_name IS NULL THEN 1 ELSE NULL END) AS null_count
FROM (
    SELECT 'cpu_usage' as column_name UNION ALL
    SELECT 'memory_usage' UNION ALL
    SELECT 'network_in' UNION ALL
    SELECT 'network_out' UNION ALL
    SELECT 'disk_usage'
) x
LEFT JOIN metrics_feature mf ON 1=0
GROUP BY column_name;
```

**Example input / output:**

```text
-- Input
SELECT
    column_name,
    COUNT(CASE WHEN column_name IS NULL THEN 1 ELSE NULL END) AS null_count
FROM (
    SELECT 'cpu_usage' as column_name UNION ALL
    SELECT 'memory_usage' UNION ALL
    SELECT 'network_in' UNION ALL
    SELECT 'network_out' UNION ALL
    SELECT 'disk_usage'
) x
LEFT JOIN metrics_feature mf ON 1=0
GROUP BY column_name;

-- Output
+------------+-----------+
| column_name | null_count|
+------------+-----------+
| cpu_usage   | 1         |
| memory_usage| 1         |
| network_in  | 0         |
| network_out | 0         |
| disk_usage  | 1         |
+------------+-----------+
```

**Data:**

```sql
-- Sample Data
CREATE TABLE metrics_feature (
    id INT,
    cpu_usage DECIMAL,
    memory_usage DECIMAL,
    network_in BIGINT,
    network_out BIGINT,
    disk_usage DECIMAL
);

INSERT INTO metrics_feature (id, cpu_usage, memory_usage, network_in, network_out, disk_usage) VALUES
(1, NULL, 2.5, 3400, 1200, 60),
(2, 85.5, NULL, 2300, 900, 70),
(3, 72.0, 3.0, 3500, 1000, NULL);
```

**Flow:**

1. Yes
2. No
3. Identify Columns
4. Is Schema Dynamic?
5. Use Script to List Columns
6. Hardcode Column Names
7. Count NULLs Using SQL Query
8. Construct SQL Query withHardcoded Columns
9. Execute Query and RetrieveResults

