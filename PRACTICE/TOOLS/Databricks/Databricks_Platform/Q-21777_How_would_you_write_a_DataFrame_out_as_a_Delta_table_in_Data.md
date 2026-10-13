---
type: "practice"
difficulty: "Basic"
---

# How would you write a DataFrame out as a Delta table in Databricks?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Delta Live Tables and Serving](<../../../../TOOLS/Databricks/Databricks_Platform/Delta_Live_Tables_and_Serving.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To write a DataFrame out as a Delta table in Databricks, we need to leverage the powerful capabilities of Delta Lake provided by Databricks. The key idea is to use Databricks' built-in functions and commands to ensure data integrity and efficient storage management.

#### Triage

Question Breakdown: The question asks for writing a DataFrame as a Delta table in Databricks. This involves understanding how Delta tables work, the syntax for saving DataFrames to Delta, and potential best practices.
What's Being Probed: Interviewers are likely assessing your knowledge of Databricks' Delta Lake capabilities, familiarity with Spark operations, and understanding of data lifecycle management.

From First Principles:

Understanding Delta Tables:
Delta tables provide ACID transactions, schema enforcement, time travel, and fine-grained access control.
Saving DataFrame as Delta Table:
We can use the write method on a DataFrame to save it directly into a Delta table.

Detailed Steps:

Import necessary libraries.
PYTHON
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("DeltaExample").getOrCreate()
Load or create your DataFrame. For example, let's assume we have a DataFrame df with some data.
Write the DataFrame to Delta format.
PYTHON
# Example DataFrame creation for demonstration
data = [("James", "Sales", 3000),
        ("Michael", "Sales", 4600),
        ("Robert", "Sales", 4100),
        ("Maria", "Finance", 3000)]
columns = ["Name", "Department", "Salary"]
df = spark.createDataFrame(data, schema=columns)
Write the DataFrame to a Delta table.
PYTHON
df.write.format("delta").save("/path/to/delta/table")
Here, /path/to/delta/table is where you want your Delta table to be stored.
Optional: Set up table properties for more control over schema enforcement and other features.
PYTHON
(df.write
 .format("delta")
 .option("overwriteTable", "true")  # Overwrite the existing table if it exists
 .mode("append")  # Append data to an existing table, or create a new one if it doesn't exist
 .save("/path/to/delta/table"))

#### Closing Summary
To write a DataFrame as a Delta table in Databricks, we use Spark's DataFrame API with the write.format("delta").save(path) method. This ensures that our data is stored efficiently and managed according to Delta Lake rules.

#### Common Mistakes/Follow-Ups

Using the wrong format or method (e.g., using csv instead of delta).
Forgetting to set up properties like overwriteTable or mode.
Not understanding the implications of different modes (append, overwrite, etc.).

