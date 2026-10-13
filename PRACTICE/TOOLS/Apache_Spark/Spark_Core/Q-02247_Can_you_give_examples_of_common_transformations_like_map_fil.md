---
type: "practice"
difficulty: "Basic"
---

# Can you give examples of common transformations like map filter and flatMap?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DataFrames and Datasets](<../../../../TOOLS/Apache_Spark/Spark_Core/DataFrames_and_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Apache Spark is a powerful framework used for large-scale data processing, and it provides several built-in operations to manipulate datasets efficiently. Common transformations in Apache Spark include map, filter, and flatMap. These are fundamental operations that you'll often use when working with DataFrames or RDDs (Resilient Distributed Datasets).

First, let’s understand what each of these transformations does:

map: Applies a function to each element of the dataset.
filter: Filters out elements from the dataset based on a condition.
flatMap: Similar to map, but it also flattens the resulting RDD.

Now, let’s dive into some examples for each transformation using Python and Spark SQL:

#### Triaging the Question

The question is asking for practical examples of these transformations. The interviewer might be probing your understanding of how to apply these operations in real-world scenarios and your familiarity with the syntax and semantics of these functions.

#### First-Principles Solution

#### Example
1: Using map

Let’s say we have a list of integers, and we want to square each number:

PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("SparkTransformationsExample").getOrCreate()

# Sample data
data = [1, 2, 3, 4]

# Create a DataFrame from the list
df = spark.createDataFrame(data, "integer")

# Apply map transformation to square each number
squared_df = df.rdd.map(lambda x: (x, x ** 2))

# Convert back to DataFrame and display
result_squares = squared_df.toDF(["original", "squared"])
result_squares.show()

#### Example
2: Using filter

Suppose we want to filter out even numbers from our dataset:

PYTHON
# Apply filter transformation to keep only odd numbers
filtered_df = df.rdd.filter(lambda x: x % 2 != 0)

# Convert back to DataFrame and display
result_filtered = filtered_df.toDF(["odd"])
result_filtered.show()

#### Example
3: Using flatMap

Imagine we have a list of strings, where each string contains multiple words. We want to split each string into individual words:

PYTHON
# Sample data with multiple words in a single column
data_with_words = ["apple orange", "banana grape", "cherry melon"]

# Create a DataFrame from the list
df_words = spark.createDataFrame(data_with_words, "string")

# Apply flatMap transformation to split each string into individual words
split_df = df_words.rdd.flatMap(lambda x: x.split(" ")).map(lambda word: (word, 1))

# Convert back to DataFrame and display
result_split = split_df.toDF(["word", "count"])
result_split.show()

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, we've seen how map, filter, and flatMap can be used in Apache Spark to transform data. Remember that these transformations are lazy by default (they only execute when an action is called), which makes them efficient for large datasets.

#### Common mistakes
include not correctly defining the transformation function or forgetting to convert RDDs back to DataFrames when displaying results.

If asked follow-ups, you might be questioned about the performance implications of each transformation, how these operations interact with RDD lineage and persistence strategies, and their use in complex data processing pipelines.

**Comparison:**

| original | squared |
|----------|---------|
|      1   |    1    |
|      2   |    4    |
|      3   |    9    |
|      4   |   16    |
| odd     |
|---------|
|    1    |
|    3    |
| word   | count |
|--------|-------|
| apple  |   1   |
| orange |   1   |
| banana |   1   |
| grape  |   1   |
| cherry |   1   |
| melon  |   1   |

**Code:**

```python
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("SparkTransformationsExample").getOrCreate()

# Sample data for map example
data_map = [1, 2, 3, 4]
df_map = spark.createDataFrame(data_map, "integer")

# Apply map transformation to square each number
squared_df_map = df_map.rdd.map(lambda x: (x, x ** 2))
result_squares_map = squared_df_map.toDF(["original", "squared"])
result_squares_map.show()

# Sample data for filter example
data_filter = [1, 2, 3, 4]
df_filter = spark.createDataFrame(data_filter, "integer")

# Apply filter transformation to keep only odd numbers
filtered_df_filter = df_filter.rdd.filter(lambda x: x % 2 != 0)
result_filtered_filter = filtered_df_filter.toDF(["odd"])
result_filtered_filter.show()

# Sample data for flatMap example
data_flatMap = ["apple orange", "banana grape", "cherry melon"]
df_flatMap = spark.createDataFrame(data_flatMap, "string")

# Apply flatMap transformation to split each string into individual words
split_df_flatMap = df_flatMap.rdd.flatMap(lambda x: x.split(" ")).map(lambda word: (word, 1))
result_split_flatMap = split_df_flatMap.toDF(["word", "count"])
result_split_flatMap.show()
```

**Example input / output:**

```text
+-------+--------+
|original|squared |
+-------+--------+
|      1|       1|
|      2|       4|
|      3|       9|
|      4|      16|
+-------+--------+

+----+
| odd|
+----+
|   1|
|   3|
+----+

+-----+------+
| word|count |
+-----+------+
|apple|     1|
|orange|     1|
|banana|     1|
|grape |     1|
|cherry|     1|
|melon|     1|
+-----+------+
```

**Data:**

```json
{
    "original": [1, 2, 3, 4],
    "squared": [1, 4, 9, 16]
}
{
    "odd": [1, 3]
}
{
    "word": ["apple", "orange", "banana", "grape", "cherry", "melon"],
    "count": [1, 1, 1, 1, 1, 1]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Yes
6. Initialize Spark Session
7. Is the data in a DataFrame?
8. Create DataFrame from List
9. Apply `map` Transformation toSquare Each Number
10. Convert back to DataFrameand Display Result
11. Is it an RDD?
12. Apply `filter` Transformation toKeep Odd Numbers
13. Convert back to DataFrameand Display Result
14. Is it a flatMap Example?
15. Apply `flatMap` Transformationto Split Words
16. Convert back to DataFrameand Display Result

