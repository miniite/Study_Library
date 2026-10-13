---
type: "practice"
difficulty: "Intermediate"
---

# What is the difference between a transformation and an action in PySpark, and can you give an example of each in Python?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pandas vs Polars vs PySpark](<../../../../TOOLS/Python/Python_Language/Pandas_vs_Polars_vs_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In PySpark, transformations and actions are two fundamental operations that define how data is processed and manipulated within Spark's Resilient Distributed Dataset (RDD) or DataFrame/DataSet APIs. A transformation creates a new RDD/DataFrame/DataSet from the current one but does not compute the result; it only builds up an execution plan. An action computes the actual results, causing the job to be executed.

To illustrate these concepts in Python, let's consider a simple example using PySpark:

#### Triaging the Question

The core of this question is to understand and differentiate between transformation operations (like map, filter, etc.) and action operations (like collect, count, etc.). The interviewer might be probing your understanding of lazy evaluation in Spark, how transformations accumulate operations without executing them immediately, and when actions trigger the actual computation.

#### First-Principles Solution
Transformation: This is an operation that returns a new RDD/DataFrame/DataSet but does not execute any computation immediately. It only builds up the lineage of operations which can be optimized by Spark's execution engine.
Action: This is an operation that triggers the actual computation and returns a result to the driver program, such as collecting data or counting elements.

#### Example
in Python

Let's walk through an example where we use both transformations and actions:

PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("TransformationVsActionExample").getOrCreate()

# Sample data: create a DataFrame with some sample data
data = [("James", "Sales", 3000), ("Michael", "Sales", 4600), ("Robert", "Sales", 4100),
        ("Maria", "Finance", 3000), ("James", "Sales", 3000), ("Scott", "Finance", 3800),
        ("Jen", "Finance", 3900)]
columns = ["employee_name", "department", "salary"]
df = spark.createDataFrame(data, columns)

# Transformation example: Filter rows where salary is greater than 4000
filtered_df = df.filter(df["salary"] > 4000)

# Action example: Collect the filtered data into a Python list and print it
result = filtered_df.collect()
for row in result:
    print(row)

#### Closing Summary
and Common Mistakes/Follow-Ups

In this example, filter is a transformation because it returns a new DataFrame but does not immediately compute anything. The actual computation happens when we call the action collect, which triggers Spark to execute the job and return results.

A common mistake beginners make is thinking that transformations themselves trigger computations. Always remember that transformations only build up an execution plan, while actions are what actually cause the computation to happen.

#### Follow-up
questions might include:

How can you optimize your transformations for better performance?
Can you explain how lineage works in PySpark and why it's important?

**Code:**

```python
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("TransformationVsActionExample").getOrCreate()

# Sample data: create a DataFrame with some sample data
data = [("James", "Sales", 3000), ("Michael", "Sales", 4600), ("Robert", "Sales", 4100),
        ("Maria", "Finance", 3000), ("James", "Sales", 3000), ("Scott", "Finance", 3800),
        ("Jen", "Finance", 3900)]
columns = ["employee_name", "department", "salary"]
df = spark.createDataFrame(data, columns)

# Transformation example: Filter rows where salary is greater than 4000
filtered_df = df.filter(df["salary"] > 4000)

# Action example: Collect the filtered data into a Python list and print it
result = filtered_df.collect()
for row in result:
    print(row)
```

**Example input / output:**

```text
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("TransformationVsActionExample").getOrCreate()

# Sample data: create a DataFrame with some sample data
data = [("James", "Sales", 3000), ("Michael", "Sales", 4600), ("Robert", "Sales", 4100),
        ("Maria", "Finance", 3000), ("James", "Sales", 3000), ("Scott", "Finance", 3800),
        ("Jen", "Finance", 3900)]
columns = ["employee_name", "department", "salary"]
df = spark.createDataFrame(data, columns)

# Transformation example: Filter rows where salary is greater than 4000
filtered_df = df.filter(df["salary"] > 4000)

# Action example: Collect the filtered data into a Python list and print it
result = filtered_df.collect()
```

