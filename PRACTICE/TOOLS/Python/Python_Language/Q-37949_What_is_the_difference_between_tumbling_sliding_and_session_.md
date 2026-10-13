---
type: "practice"
difficulty: "Deep"
---

# What is the difference between tumbling, sliding, and session windows in stream processing, and how would you express a 5-minute tumbling aggregation in PySpark?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Large Data Migrations in Python](<../../../../TOOLS/Python/Python_Language/Large_Data_Migrations_in_Python.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with streaming data in Apache Spark, understanding window operations is crucial. Windows are used to group events over a specified time period or based on certain conditions. In this context, we'll explore tumbling, sliding, and session windows, and then delve into how to implement a 5-minute tumbling aggregation using PySpark.

Tumbling, sliding, and session windows serve different purposes in stream processing:

Tumbling Windows: These are non-overlapping intervals of time where each event is processed independently. For example, if you set a window size of 5 minutes, every 5 minutes a new batch starts from the beginning, processing all events that arrived during that period.

Sliding Windows: In contrast to tumbling windows, sliding windows overlap. They also process events in fixed-size intervals but slide over time, allowing for overlapping batches. This means if you set a window size of 5 minutes with a slide interval of 3 minutes, every 3 minutes a new batch starts, capturing both the current and previous 2-minute periods.

Session Windows: These windows are based on events occurring within a session where events are grouped by gaps between them. A typical use case is to treat events as part of the same "session" if they occur close together (e.g., within 5 minutes). If there's no event for more than 5 minutes, the current session ends and a new one begins.

To illustrate these concepts in PySpark, let's focus on implementing a 5-minute tumbling aggregation. We'll start by setting up our environment and sample data.

**Comparison:**

| window          | count |
|-----------------|-------|
| [2023-09-01 10:05, 2023-09-01 10:10) | 2     |
| [2023-09-01 10:10, 2023-09-01 10:15) | 1     |
| [2023-09-01 10:15, 2023-09-01 10:20) | 2     |

**Code:**

```python
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder \
    .appName("Window Operations Example") \
    .getOrCreate()

# Sample data in a DataFrame
data = [
    (1, "2023-09-01 10:05:00"),
    (2, "2023-09-01 10:07:00"),
    (3, "2023-09-01 10:15:00"),
    (4, "2023-09-01 10:16:00"),
    (5, "2023-09-01 10:25:00")
]
columns = ["id", "timestamp"]
df = spark.createDataFrame(data, columns)
df.show()
```

**Example input / output:**

```

We'll convert the timestamp column to a timestamp type and then use the window function from PySpark's SQL functions to define our tumbling window.

PYTHON
Copy
from pyspark.sql.functions import col, expr

# Convert timestamp to proper format
df = df.withColumn("timestamp", col("timestamp").cast("timestamp"))

# Define tumbling window with a size of 5 minutes
window_spec = expr("window(timestamp, '5 minutes')")

# Group by the window and aggregate (assuming we want to count occurrences)
result_df = df.groupBy(window_spec).count()

# Show the result
result_df.show()
```

**Data:**

```plaintext
+---+---------------------+
| id|             timestamp|
+---+---------------------+
|  1|2023-09-01 10:05:00   |
|  2|2023-09-01 10:07:00   |
|  3|2023-09-01 10:15:00   |
|  4|2023-09-01 10:16:00   |
|  5|2023-09-01 10:25:00   |
+---+---------------------+
```

**Flow:**

1. Input DataFrame
2. Is timestamp column in properformat?
3. C
4. Create window specificationwith size of 5 minutes
5. Define tumbling window using`window` function
6. Group by the window andaggregate count
7. Output aggregated results

