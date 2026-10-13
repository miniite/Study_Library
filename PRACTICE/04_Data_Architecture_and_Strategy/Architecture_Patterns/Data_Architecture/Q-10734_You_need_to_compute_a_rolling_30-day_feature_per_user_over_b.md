---
type: "practice"
difficulty: "Deep"
---

# You need to compute a rolling 30-day feature per user over billions of events. How does the lake's partitioning and file layout make or break this job?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Tiered Lake Storage](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/_index.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compute a rolling 30-day feature per user over billions of events, understanding how the data lake is structured—specifically its partitioning and file layout—is crucial. This setup significantly influences performance, storage efficiency, and ease of querying large datasets. In this scenario, we need to leverage advanced partitioning techniques such as time-based or user-based partitions to optimize our rolling feature computation.

#### Triaging the Question

The core question here is: How does the data lake's structure impact the ability to efficiently compute a rolling 30-day window per user? The interviewer is probing your understanding of data architecture best practices, particularly in handling large-scale data. Assumptions include that we are dealing with a large volume of events and need to perform aggregations or transformations on these events.

#### First Principles Solution
Partitioning Strategy: We need to choose an effective partitioning strategy that minimizes the amount of data scanned during our query execution.
File Layout: The file layout should support efficient data access and querying, often using a format like Parquet for its columnar storage benefits.
Query Optimization: Leveraging advanced features such as SQL over distributed databases or frameworks like Apache Spark can help achieve the desired performance.
Detailed Solution
Partitioning Strategy

Partitioning by user_id and event_date would be an excellent strategy here:

User-Based Partitioning (user_id): This ensures that each user’s events are stored together, reducing data shuffling during queries.
Time-Based Partitioning (event_date or window_start_date): Breaking down the event stream into daily partitions helps in managing and querying large datasets more efficiently. For a rolling 30-day window, we might further partition by month or week to reduce scanning time.
File Layout

Using columnar storage formats like Parquet is highly beneficial:

Columnar Storage: Reduces I/O operations since only relevant columns are read during queries.
Encoding Schemes: Utilizing efficient encoding schemes within Parquet can further enhance performance, especially for filtering and sorting tasks.
Query Optimization

Leveraging distributed computing frameworks (e.g., Apache Spark) with appropriate configurations is essential:

Window Functions: Using window functions in SQL to define the 30-day rolling period.
Caching Intermediate Results: Caching results of expensive subqueries can significantly speed up repeated queries.

#### Closing Summary
and Common Mistakes

To efficiently compute a rolling 30-day feature per user, we need a carefully designed data architecture that balances partitioning strategies with file layouts. Poorly structured data lakes can lead to performance bottlenecks and increased storage costs. Key mistakes include:

Inadequate Partitioning: Failing to partition by both user_id and event_date, leading to excessive data scanning.
Incorrect File Formats: Choosing less efficient formats (e.g., CSV) that do not support columnar access patterns.
Common Follow-Ups
How would you handle varying event frequencies?
What are the trade-offs between different partitioning strategies?
How would you ensure data quality and integrity in this setup?

**Comparison:**

```
user_id	event_date	feature_value
u1	2023-10-01	10
u2	2023-10-05	5
u1	2023-10-06	7
user_id	window_start_date	rolling_sum
u1	2023-10-01	17
```

**Code:**

```python
# Example code using PySpark for rolling window feature computation

from pyspark.sql import SparkSession
from pyspark.sql.functions import col, window, sum as spark_sum

spark = SparkSession.builder.appName("RollingWindowFeature").getOrCreate()

events_df = spark.read.parquet("/path/to/data")
users_df = events_df.groupBy("user_id", window(col("event_time"), "30 days")).agg(spark_sum("feature_column"))
```

**Example input / output:**

```text
Input:
user_id | event_date | feature_value
u1      | 2023-10-01 | 10
u2      | 2023-10-05 | 5
u1      | 2023-10-06 | 7

Output:
user_id | window_start_date | rolling_sum
u1      | 2023-10-01        | 17 (10 + 7)
```

**Data:**

```json
{
  "user_id": ["u1", "u2", "u1"],
  "event_date": ["2023-10-01", "2023-10-05", "2023-10-06"],
  "feature_value": [10, 5, 7]
}
```

**Flow:**

1. Data Lake
2. Partition by User ID
3. Partition by Event Date
4. Store as Parquet
5. Use Spark for Aggregation
6. Output Rolling Sum per User

