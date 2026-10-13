---
type: "practice"
difficulty: "Basic"
---

# How do you filter rows in a DataFrame using the filter or where method?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DataFrames and Datasets](<../../../../TOOLS/Apache_Spark/Spark_Core/DataFrames_and_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: Filtering rows in a DataFrame using Apache Spark involves specifying conditions that define which rows should be retained based on certain column values. The filter and where methods are essentially interchangeable, both allowing you to apply these conditions.

#### Triage
This question aims to test your understanding of basic DataFrame operations in Apache Spark. We assume the context is working with a large dataset stored as a DataFrame and that you need to extract specific rows based on some criteria. Common mistakes include not fully understanding the syntax or mistaking filter for where.

To filter rows, we start from the fundamental concept: specifying conditions on columns to retain only those rows that meet these conditions.

Step-by-Step Solution: Let's assume we have a DataFrame with multiple columns and we want to filter it based on certain conditions. For example, consider a DataFrame containing user activity logs where each row represents an interaction by a user. We might want to keep only the interactions for users who logged in more than 10 times.

PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("DataFrameFilterExample").getOrCreate()

# Sample DataFrame schema (assuming columns: 'userId', 'loginCount')
data_schema = "userId INT, loginCount INT"
data = [("user1", 5), ("user2", 10), ("user3", 15)]
df = spark.createDataFrame(data, data_schema)

To filter users who logged in more than 10 times, we use the filter or where method:

PYTHON
# Using filter
filtered_df = df.filter("loginCount > 10")

# Alternatively, using where (same as filter for this case)
filtered_df = df.where("loginCount > 10")

Both methods will result in a new DataFrame containing only the rows where loginCount is greater than 10.

#### Closing Summary
The filter and where methods in Apache Spark are used to retain rows based on specified conditions. They can be applied directly to DataFrames, offering flexibility and ease of use for filtering operations. It's important to understand that they essentially do the same thing, and choosing one over the other is often a matter of personal preference or stylistic choice.

#### Common Mistakes/Follow-Ups
Common pitfalls include misinterpreting the syntax or conditions in complex scenarios. For example, ensuring you are using the correct column names and understanding boolean expressions fully. A follow-up question might explore more complex filtering scenarios involving multiple columns or nested conditions.

**Comparison:**

| userId | loginCount |
|--------|------------|
| user1  |     5      |
| user2  |    10      |
| user3  |    15      |
| userId | loginCount |
|--------|------------|
| user3  |    15      |

**Code:**

```python
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("DataFrameFilterExample").getOrCreate()

# Sample DataFrame schema (assuming columns: 'userId', 'loginCount')
data_schema = "userId INT, loginCount INT"
data = [("user1", 5), ("user2", 10), ("user3", 15)]
df = spark.createDataFrame(data, data_schema)

# Using filter
filtered_df = df.filter("loginCount > 10")

# Alternatively, using where (same as filter for this case)
filtered_df.where("loginCount > 10")
```

**Example input / output:**

```text
Input DataFrame:
userId | loginCount
-----------------
user1  |     5
user2  |    10
user3  |    15

Filtered DataFrame using `filter` or `where`:
userId | loginCount
-----------------
user3  |    15
```

**Data:**

```plaintext
userId | loginCount
-----------------
user1  |     5
user2  |    10
user3  |    15
```

**Flow:**

1. DataFrame Creation
2. Using 'filter' or 'where'?
3. Apply Condition and GetFiltered DataFrame
4. Return Filtered DataFrame

