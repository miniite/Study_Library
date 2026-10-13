---
type: "practice"
difficulty: "Basic"
---

# How do you remove duplicate rows from a DataFrame in Spark?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DataFrames and Datasets](<../../../../TOOLS/Apache_Spark/Spark_Core/DataFrames_and_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To remove duplicate rows from a DataFrame in Apache Spark, we first need to understand that duplicates can be identified based on one or more columns. The core concept here is to use Spark's built-in functions and operations to filter out these duplicates efficiently. Let's walk through this step-by-step.

Firstly, it’s important to know that Spark handles data in a distributed manner across multiple nodes, which makes efficient duplicate detection and removal a bit different from traditional SQL-based solutions. The key idea is to use the dropDuplicates() method provided by the DataFrame API.

Now, let's dive into the process with some assumptions:

We are working with a DataFrame that contains duplicate rows based on certain columns.
Our goal is to keep only one instance of each unique row and remove any redundant entries.

Here’s how we can achieve this:

Identify the Columns: Determine which columns define uniqueness in your data. For example, let's assume our DataFrame has columns id, name, and value.
Use dropDuplicates() Method: This method will identify duplicates based on the specified column(s) or all columns if no argument is provided.
Check for Performance Considerations: Depending on the size of your data, consider partitioning and shuffling to optimize performance.

Let’s see a practical example:

PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("RemoveDuplicates").getOrCreate()

# Sample DataFrame creation (for demonstration)
data = [
    (1, "Alice", 30),
    (2, "Bob", 40),
    (2, "Bob", 50),  # Duplicate entry
    (3, "Charlie", 60),
]

columns = ["id", "name", "value"]
df = spark.createDataFrame(data, columns)

# Display the original DataFrame
print("Original DataFrame:")
df.show()

# Remove duplicates based on all columns
unique_df = df.dropDuplicates()

# Display the result
print("DataFrame after removing duplicates:")
unique_df.show()

In this example, dropDuplicates() is used without specifying any column names, so it will consider all columns. If you want to specify certain columns, you can do so:

PYTHON
unique_df = df.dropDuplicates(["id", "name"])

This ensures only unique combinations of id and name are retained.

**Comparison:**

| id | name   | value |
|----|--------|-------|
| 1  | Alice  | 30    |
| 2  | Bob    | 40    |
| 3  | Charlie| 60    |

**Code:**

```python
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("RemoveDuplicates").getOrCreate()

# Sample DataFrame creation (for demonstration)
data = [
    (1, "Alice", 30),
    (2, "Bob", 40),
    (2, "Bob", 50),  # Duplicate entry
    (3, "Charlie", 60),
]

columns = ["id", "name", "value"]
df = spark.createDataFrame(data, columns)

# Display the original DataFrame
print("Original DataFrame:")
df.show()

# Remove duplicates based on all columns
unique_df = df.dropDuplicates()

# Display the result
print("DataFrame after removing duplicates:")
unique_df.show()
```

**Example input / output:**

```

Original DataFrame:

PLAINTEXT
Copy
+---+-------+----+
| id|   name|value|
+---+-------+----+
|  1| Alice |  30|
|  2|   Bob |  40|
|  2|   Bob |  50|  # Duplicate entry
|  3|Charlie|  60|
+---+-------+----+

DataFrame after removing duplicates:

PLAINTEXT
Copy
+---+-------+----+
| id|   name|value|
+---+-------+----+
|  1| Alice |  30|
|  2|   Bob |  40|
|  3|Charlie|  60|
+---+-------+----+
```

**Data:**

```python
data = [
    (1, "Alice", 30),
    (2, "Bob", 40),
    (2, "Bob", 50),  # Duplicate entry
    (3, "Charlie", 60),
]

columns = ["id", "name", "value"]
```

**Flow:**

1. Initiate Spark Session
2. Create DataFrame
3. Define Columns
4. Add Sample Data
5. Display Original DataFrame
6. Use dropDuplicates()
7. Display Result

