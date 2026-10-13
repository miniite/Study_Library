---
type: "practice"
difficulty: "Intermediate"
---

# Explain what time-travel gives you in Iceberg or Delta and a real situation where it saved you. What does keeping those snapshots cost in storage?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Lake Storage](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Time-travel capabilities, such as those provided by Apache Iceberg and Databricks Delta, allow you to access past states of your data at any point in time. This is incredibly valuable for auditing, debugging, and performing complex queries without needing to re-run all the transformations from scratch. In my experience, this feature has saved us significant time when we needed to debug a production issue that required understanding previous states of our dataset.

For example, during a recent incident where we experienced an unexpected data drift in one of our key metrics, we were able to use Iceberg’s time-travel feature to pinpoint exactly when and why the metric deviated from expected values. By querying historical snapshots of the table, we could trace back through each change made and identify the specific update that caused the issue. This allowed us to quickly understand the root cause and fix the problem without disrupting ongoing operations.

Time-travel comes at a cost in terms of storage overhead. Each snapshot represents an additional copy of your data, which can significantly increase the amount of space required on disk or cloud storage. However, the trade-off is often worth it due to the benefits gained from being able to analyze historical states of your data without rewriting large portions of your ETL pipeline.

**Code:**

```python
# Example Python code using Databricks Delta's time-travel feature
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("TimeTravelExample").getOrCreate()

# Load the table with a specific snapshot version
df = spark.read.format("delta").option("versionAsOf", "2023-10-05T14:30:00Z").load("/path/to/table")

# Perform analysis on historical data
df.show()
```

**Example input / output:**

```text
No input-output examples are necessary here, as the focus is on explaining concepts rather than providing concrete outputs.
```

**Data:**

```

No specific schemas or JSON samples are needed for this answer, but in general, you might have a table schema like:

SQL
Copy
CREATE TABLE metrics (
    metric_name STRING,
    value DOUBLE,
    timestamp TIMESTAMP,
    -- other columns ...
) USING DELTA;
```

**Flow:**

1. Yes
2. No
3. Identify Issue
4. Use Time-Travel?
5. Query Historical Snapshots
6. Possibly Identify Root CauseQuickly
7. Solve Problem Efficiently
8. Re-run ETL Pipeline fromScratch
9. May Take Longer to SolveIssue

