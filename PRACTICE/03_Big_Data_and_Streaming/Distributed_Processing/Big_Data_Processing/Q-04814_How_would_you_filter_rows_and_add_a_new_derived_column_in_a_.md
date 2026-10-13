---
type: "practice"
difficulty: "Basic"
---

# How would you filter rows and add a new derived column in a Spark DataFrame during a simple ETL step?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Spark DataFrames](<../../../../03_Big_Data_and_Streaming/Distributed_Processing/Big_Data_Processing/Spark_DataFrames.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In this scenario, we are tasked with performing an ETL (Extract, Transform, Load) step using Apache Spark to filter rows based on certain conditions and add a new derived column in a DataFrame. This operation is fundamental in data engineering as it helps ensure that the processed data meets specific criteria before loading into a database or another system.

To break down the question:

Filtering Rows: We need to select only those rows that meet certain conditions.
Adding a New Derived Column: We must create a new column based on existing columns' values.

Let's walk through this from first principles:

Step 1: Load and Prepare Data

First, we load the data into a Spark DataFrame. For simplicity, let’s assume our input DataFrame contains customer information with fields like customer_id, purchase_date, amount.

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("ETLExample").getOrCreate()
df = spark.read.csv("/path/to/input/file", header=True, inferSchema=True)
Step 2: Filter Rows

We will filter the rows based on a condition. For example, let’s say we want to keep only those customers who made purchases after January 1, 2023.

PYTHON
filtered_df = df.filter(df["purchase_date"] > "2023-01-01")
Step 3: Add Derived Column

Next, we need to add a new column. Let’s say the derived column will be is_eligible, which is true if the purchase amount is greater than $50.

PYTHON
from pyspark.sql.functions import col

filtered_df = filtered_df.withColumn("is_eligible", col("amount") > 50)
Step 4: Show Resulting DataFrame

Finally, we can display the resulting DataFrame to ensure our operations were performed correctly.

PYTHON
filtered_df.show()

#### Closing Summary
and Common Mistakes/Follow-Ups

In summary, we loaded a CSV file into a Spark DataFrame, filtered out rows based on a condition, added a derived column indicating whether the purchase amount was eligible, and displayed the resulting DataFrame. Key points to remember include:

Proper Data Loading: Ensure the data is correctly read and understood.
Efficient Filtering: Use appropriate conditions for filtering to avoid unnecessary processing.
Derived Columns: Carefully define the logic for creating new columns to ensure correctness.

#### Common mistakes
might include:

Incorrectly defining filters, leading to unexpected rows being retained or discarded.
Failing to handle data types correctly when adding derived columns, causing errors like type mismatches.

For more complex scenarios, consider handling missing values and edge cases.

