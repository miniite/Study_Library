---
type: "practice"
difficulty: "Basic"
---

# What is the difference between a narrow and a wide transformation and why does that distinction determine your job's number of stages?

**Difficulty:** Basic · **Levels:** SDE-1/2, SDE-2/3 · [↑ Reducing Shuffle in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Reducing_Shuffle_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In Data Engineering, particularly when working with PySpark, understanding the distinction between narrow and wide transformations is crucial. A narrow transformation operates on each partition independently without shuffling data across partitions, whereas a wide transformation requires shuffling of data to distribute tasks across partitions. The key difference determines how many stages your job will be split into, which impacts performance.

To begin, let’s triage the question: The interviewer is probing your understanding of Spark's execution model and its impact on job optimization. We are assuming you have experience with PySpark and are familiar with the terms "narrow" and "wide" transformations, as well as stages in a Spark job.

Narrow Transformations

Narrow transformations include map, filter, sample, and distinct. These operations can be performed on each partition of data independently without needing to shuffle the data. For example, if you apply a filter operation on a DataFrame, it only affects the local partition and does not require redistributing any other partitions.

Wide Transformations

Wide transformations include groupByKey, reduceByKey, join, and cogroup. These operations involve shuffling the entire dataset to distribute tasks across different partitions. For instance, when you join two DataFrames, Spark will first redistribute (shuffle) both datasets before performing the join operation.

Stages in a Spark Job

In PySpark, a stage is a set of tasks that can be executed independently. The number of stages directly impacts how parallelizable your job can be and thus its overall performance. Narrow transformations are often used to create multiple stages because they allow for more fine-grained parallelism. Wide transformations typically define fewer stages since the data must first be shuffled, limiting the number of independent tasks.

Step-by-Step Solution
Identify Transformation Type: Determine if a transformation is narrow or wide.
Shuffling Data: Narrow transformations do not require shuffling (no data movement between partitions). Wide transformations necessitate shuffling to distribute the load across multiple nodes.
Stage Creation: Each stage in PySpark corresponds to a single transformation, but only wide transformations can trigger new stages due to the necessity of shuffling.

#### Example

#### Cons
ider a simple ETL process using PySpark:

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ETLExample").getOrCreate()

# Load data
data1 = spark.read.csv("path/to/data1")
data2 = spark.read.csv("path/to/data2")

# Narrow transformation: filter
filtered_data1 = data1.filter(data1.column_name > 5)

# Wide transformation: join
joined_data = filtered_data1.join(data2, "common_key", "inner")

In this example, the filter operation on data1 is a narrow transformation, whereas the join operation is wide. The filter will create one stage, and the join will create another because it requires shuffling.

#### Closing Summary

The distinction between narrow and wide transformations in PySpark fundamentally determines how your job's stages are created. Narrow transformations do not trigger new stages, while wide transformations often necessitate them due to data shuffling requirements. Understanding this helps optimize Spark jobs for better performance by leveraging the parallelism potential of each transformation type.

#### Common Mistakes/Follow-Ups
Mistake: Overusing wide transformations in an ETL pipeline can lead to unnecessary stages and reduced parallelism.

#### Follow-Up
How do you balance between narrow and wide transformations when designing a large-scale data processing job? Consider the trade-offs in terms of memory, network bandwidth, and overall job execution time.

**Comparison:**

```
Yes
No
Load data1
Is filter narrow?
Create stage for filter
Load data2 and join withfiltered_data1
Is join wide?
Yes --> G[Create new stage forjoin]
Continue processing in samestage
No narrow transformationcreates a new stage
```

**Code:**

```python
# Example code snippet for clarity
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ETLExample").getOrCreate()

# Load data
data1 = spark.read.csv("path/to/data1")
data2 = spark.read.csv("path/to/data2")

# Narrow transformation: filter
filtered_data1 = data1.filter(data1.column_name > 5)

# Wide transformation: join
joined_data = filtered_data1.join(data2, "common_key", "inner")
```

**Example input / output:**

| Input | Output |
|-------|--------|
| data1 = spark.read.csv("path/to/data1")<br>data2 = spark.read.csv("path/to/data2")<br>filtered_data1 = data1.filter(data1.column_name > 5)<br>joined_data = filtered_data1.join(data2, "common_key", "inner") | joined_data is a DataFrame with the following rows:<br>| common_key | column_name | related_data |
|---------|------------|-------------|
| Y       | 7          | 2           |

**Data:**

```json
{
    "data1": {
        "columns": ["column_name", "other_columns"],
        "rows": [
            {"column_name": 3, "other_columns": "A"},
            {"column_name": 7, "other_columns": "B"}
        ]
    },
    "data2": {
        "columns": ["common_key", "related_data"],
        "rows": [
            {"common_key": "X", "related_data": "1"},
            {"common_key": "Y", "related_data": "2"}
        ]
    }
}
```

**Flow:**

1. Yes
2. No
3. Load data1
4. Is filter narrow?
5. Create stage for filter
6. Load data2 and join withfiltered_data1
7. Is join wide?
8. Yes --> G[Create new stage forjoin]
9. Continue processing in samestage
10. No narrow transformationcreates a new stage

