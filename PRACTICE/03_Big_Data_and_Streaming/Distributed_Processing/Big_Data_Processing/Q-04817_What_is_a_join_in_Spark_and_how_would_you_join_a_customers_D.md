---
type: "practice"
difficulty: "Basic"
---

# What is a join in Spark and how would you join a customers DataFrame with an orders DataFrame?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Spark DataFrames](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_DataFrames.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Joining DataFrames in Apache Spark is a fundamental operation used to combine two datasets based on a common key. This is akin to a database JOIN operation, but performed on large datasets using distributed processing. In this context, we'll join the "customers" DataFrame with the "orders" DataFrame using the customer ID as the common key.

#### Triaging the Question

The question is asking about the concept of joining DataFrames in Spark and how to perform a specific type of join (likely an inner join) between two DataFrames: customers and orders. The interviewer might be probing your understanding of DataFrame operations, specifically joins. We'll make some assumptions:

Both DataFrames have a common key (customer_id).
We're interested in matching customers with their orders.
Step-by-Step Solution from First Principles

First, let's import the necessary libraries and create sample DataFrames to represent our scenario:

PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.appName("DataFrameJoinExample").getOrCreate()

# Sample DataFrames for customers and orders
customers_data = [
    (1, "Alice", 30),
    (2, "Bob", 40),
    (3, "Charlie", 50)
]

orders_data = [
    (1, 200),
    (1, 300),
    (2, 400)
]

# Create DataFrames
customers_df = spark.createDataFrame(customers_data, ["customer_id", "name", "age"])
orders_df = spark.createDataFrame(orders_data, ["customer_id", "order_amount"])

# Show the DataFrames to understand their structure
customers_df.show()
orders_df.show()

The customers DataFrame contains customer IDs and names, while the orders DataFrame contains order amounts associated with those customers. To join these DataFrames on the common key customer_id, we use the join method in Spark.

PYTHON
# Perform an inner join based on 'customer_id'
joined_df = customers_df.join(orders_df, "customer_id")

# Display the result
joined_df.show()

This will produce a DataFrame where each row contains customer information and their associated order amounts. The resulting schema would look something like this:

PLAINTEXT
+----------+-----+----+---------+
|customer_id|name |age |order_amount|
+----------+-----+----+-----------+
|         1 |Alice| 30 |        200 |
|         1 |Alice| 30 |        300 |
|         2 |Bob  |40  |        400 |
+----------+-----+----+-----------+

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, joining DataFrames in Spark involves using the join method to combine rows based on a common key. The most common type of join is the inner join, but there are other types such as left outer join, right outer join, full outer join, and left semi join.

#### Common mistakes
include:

Using an incorrect key for joining.
Not understanding the difference between various join types (e.g., inner, left outer, right outer).
Ignoring performance implications when dealing with large datasets.

#### Follow-up
questions might explore your knowledge of partitioning strategies, broadcast variables, or how to handle null values in the join operation.

