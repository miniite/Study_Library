---
type: "practice"
difficulty: "Intermediate"
---

# How would you use Spark to deduplicate and clean a raw dataset before loading it to a warehouse?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ DataFrames and Datasets](<../../../../TOOLS/Apache_Spark/Spark_Core/DataFrames_and_Datasets.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this problem, we'll be using Apache Spark's powerful capabilities to deduplicate and clean a raw dataset before loading it into a data warehouse. The key steps involve identifying duplicates, handling missing or invalid data, and ensuring that the cleaned data is ready for analysis. Let's break down these steps from first principles.

Firstly, we need to understand what "deduplication" means in this context—removing records with identical primary keys or a set of unique identifiers. "Cleaning" can include various tasks like removing duplicates, correcting inconsistencies, and ensuring the data is formatted correctly before loading it into the warehouse.

We'll proceed by describing each step:

Identifying Duplicates: We use Spark’s DataFrame API to group records based on key columns and filter out duplicates.
Handling Missing Data: We impute or remove missing values as appropriate, depending on their nature.
Cleaning Inconsistent Data: We validate and correct data inconsistencies before moving it into the warehouse.

Now, let's dive deeper into each step:

Identifying Duplicates:
Step 1.1: Load the raw dataset using Spark’s DataFrame API.
SCALA
val rawData = spark.read.csv("path/to/rawdata.csv")
Handling Missing Data:
Step 2.1: Identify columns with missing values and decide on a strategy to handle them (e.g., impute, drop).
SCALA
// Dropping rows with missing values in the key column for simplicity.
val cleanedData = rawData.dropna(subset = Seq("keyColumn"))
Cleaning Inconsistent Data:
Step 3.1: Validate and correct data inconsistencies. This can involve transformations like converting data types, removing leading/trailing spaces, etc.
SCALA
import org.apache.spark.sql.functions._

val cleanedData = cleanedData.withColumn("name", trim(col("name")))
  .withColumn("amount", col("amount").cast("double"))

Finally, we write the cleaned data to a new location for loading into the warehouse:

SCALA
cleanedData.write.format("parquet").mode("overwrite").save("path/to/cleaneddata.parquet")

**Comparison:**

| Cleaned Data Example |
|----------------------|
| id | name       | amount |
|----|------------|--------|
| 101| John Doe   | 20.50  |
| 102| Jane Doe   | 23.00  |

**Code:**

```scala
// Step 1.1: Load raw dataset using Spark's DataFrame API.
val rawData = spark.read.csv("path/to/rawdata.csv")

// Step 2.1: Drop rows with missing values in the key column for simplicity.
val cleanedData = rawData.dropna(subset = Seq("keyColumn"))

// Step 3.1: Validate and correct data inconsistencies.
import org.apache.spark.sql.functions._
val cleanedData = cleanedData.withColumn("name", trim(col("name")))
                               .withColumn("amount", col("amount").cast("double"))

// Write the cleaned data to a new location for loading into the warehouse.
cleanedData.write.format("parquet").mode("overwrite").save("path/to/cleaneddata.parquet")
```

**Example input / output:**

```text
Input (rawdata.csv):
id, name, amount
101, John Doe , $20.50
102, Jane Doe, 23$
101, John Doe , 20.50

Output (cleaneddata.parquet):
+---+-----------+-------+
| id|       name|amount |
+---+-----------+-------+
|101|   John Doe|  20.50|
|102|    Jane Doe|  23.00|
+---+-----------+-------+
```

**Data:**

```json
{
  "rawdata.csv": [
    {"id": "101", "name": "John Doe ", "amount": "$20.50"},
    {"id": "102", "name": "Jane Doe", "amount": "23$"},
    {"id": "101", "name": "John Doe ", "amount": "20.50"}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Load Raw Data
4. Identify Duplicates
5. Drop or Handle Missing Values
6. Validate and Clean InconsistentData
7. Write to Warehouse
8. Filter Records
9. Deduplicate Data
10. Handle Missing or Invalid Data
11. Clean and Validate Data

