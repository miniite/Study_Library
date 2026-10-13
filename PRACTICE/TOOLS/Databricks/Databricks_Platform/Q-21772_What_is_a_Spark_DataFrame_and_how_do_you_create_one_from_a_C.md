---
type: "practice"
difficulty: "Basic"
---

# What is a Spark DataFrame and how do you create one from a CSV file in Databricks?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Databricks Query Performance](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Query_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A Spark DataFrame in Databricks is an optimized, distributed representation of a dataset that can be queried using a high-level API. It's essentially a collection of rows and columns with a defined schema. To create a Spark DataFrame from a CSV file in Databricks, you use the spark.read.csv() method.

#### Overview

In this explanation, we will delve into what a Spark DataFrame is, its key features, and how to create one using Databricks. We'll start by understanding the concept of DataFrames and their advantages over RDDs (Resilient Distributed Datasets), then move on to creating them from CSV files.

Core Concepts

Spark DataFrame:

A Spark DataFrame is a distributed collection of data organized into named columns.
It provides an API that allows for SQL-like queries using the DataFrame and Dataset APIs in Spark.
Unlike RDDs, DataFrames are strongly typed, have optimized internal representation, and provide higher-level operations.

Databricks:

Databricks is a cloud-based platform built on top of Apache Spark for data engineering and machine learning.
It offers an interactive workspace where you can write, run, and visualize your code in a collaborative environment.
Question Breakdown

The question asks us to explain what a Spark DataFrame is and how to create one from a CSV file using Databricks. The core of this question involves understanding the concept of DataFrames and knowing the specific method used to load data into them from a CSV file.

#### Assumptions
We assume you are familiar with basic concepts in Apache Spark, including RDDs.
You have access to a running Databricks cluster and are comfortable writing simple code snippets in Python or Scala.
Step-by-Step Solution

Understanding the Data:

Suppose we have a CSV file named data.csv that contains user transaction data with columns: user_id, transaction_date, amount.

Loading the CSV File into Spark DataFrame:

PYTHON
# Initialize the Spark session (if not already done)
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName('DataFrameExample').getOrCreate()

# Load data from a CSV file
df = spark.read.csv('/FileStore/tables/data.csv', header=True, inferSchema=True)

# Show the first few rows of the DataFrame
df.show()

Exploring the DataFrame:

Once loaded, we can perform various operations on the DataFrame such as filtering, transforming, and aggregating data.
PYTHON
# Filter transactions where the amount is greater than 100
filtered_df = df.filter(df['amount'] > 100)

# Show the first few rows of the filtered DataFrame
filtered_df.show()

#### Closing Summary

To summarize, a Spark DataFrame in Databricks is an efficient and powerful way to handle structured data. The spark.read.csv() method allows us to easily load CSV files into DataFrames for further processing and analysis.

#### Common Mistakes/Follow-Ups
Common Mistake: Not setting the header parameter correctly can lead to misinterpreting column names.

#### Follow-Up
Question: What happens if the CSV file has a different schema than expected? How do you handle that?
Additional Information: For more complex scenarios, consider using spark.read.options() to specify additional options like delimiter and nullValue.

**Comparison:**

| user_id | transaction_date | amount |
|---------|------------------|--------|
| 001     | 2023-10-01       | 50.0   |
| 002     | 2023-10-02       | 150.0  |
| 003     | 2023-10-03       | 75.0   |

**Code:**

```python
from pyspark.sql import SparkSession

# Initialize the Spark session (if not already done)
spark = SparkSession.builder.appName('DataFrameExample').getOrCreate()

# Load data from a CSV file
df = spark.read.csv('/FileStore/tables/data.csv', header=True, inferSchema=True)

# Show the first few rows of the DataFrame
df.show()
```

**Example input / output:**

```text
+----------+---------------+-------+
| user_id   | transaction_date | amount|
+----------+---------------+-------+
| 001       | 2023-10-01      | 50.0  |
| 002       | 2023-10-02      | 150.0 |
| 003       | 2023-10-03      | 75.0  |
+----------+---------------+-------+
```

**Data:**

```json
{
  "columns": ["user_id", "transaction_date", "amount"],
  "data": [
    {"user_id": "001", "transaction_date": "2023-10-01", "amount": 50.0},
    {"user_id": "002", "transaction_date": "2023-10-02", "amount": 150.0},
    {"user_id": "003", "transaction_date": "2023-10-03", "amount": 75.0}
  ]
}
```

**Flow:**

1. No
2. Yes
3. Yes
4. Initialize Spark Session
5. Is session initialized?
6. Create new SparkSession
7. Load CSV file into DataFrame
8. Show DataFrame content
9. Filter transactions with amount> 100
10. Show filtered DataFramecontent

