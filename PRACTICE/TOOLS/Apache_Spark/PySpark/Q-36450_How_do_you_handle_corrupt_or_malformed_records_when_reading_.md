---
type: "practice"
difficulty: "Intermediate"
---

# How do you handle corrupt or malformed records when reading JSON/CSV in PySpark — explain PERMISSIVE DROPMALFORMED and FAILFAST modes and _corrupt_record.

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in PySpark](<../../../../TOOLS/Apache_Spark/PySpark/Schema_Evolution_in_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with corrupted or malformed records during data ingestion using PySpark, it's crucial to understand how different error handling modes work—PERMISSIVE DROPMALFORMED and FAILFAST. These modes dictate how PySpark should react when encountering such records. In this answer, I'll explain these modes in detail, along with an example of how to handle malformed records using the _corrupt_record attribute.

Firstly, let's break down what these modes mean:

FAILFAST (default mode): This mode throws a PySparkSqlException as soon as PySpark encounters any malformed record. It stops processing immediately and returns control to the user with an exception.
PERMISSIVE DROPMALFORMED: In this mode, PySpark continues processing even when it encounters malformed records but will log or mark these records for later handling.

Now let's dive into a practical example:

#### Triage

The question is asking you to explain the behavior of PERMISSIVE DROPMALFORMED and FAILFAST modes in PySpark during JSON/CSV data ingestion. It also requests an explanation of how to handle corrupted records using the _corrupt_record attribute.

#### First-Principles Solution

To understand these modes, let's start with a simple example where we read a CSV file containing some malformed lines. We'll use both FAILFAST and PERMISSIVE DROPMALFORMED modes to see their differences in behavior.

Here’s the step-by-step approach:

Read the Data with FAILFAST Mode:

PYTHON
from pyspark.sql import SparkSession

spark = SparkSession.builder.getOrCreate()
try:
    df_failfast = spark.read.csv("path/to/malformed_data.csv", header=True, inferSchema=True)
except Exception as e:
    print(f"FAILFAST mode encountered an error: {e}")

Read the Data with PERMISSIVE DROPMALFORMED Mode:

PYTHON
df_permissive = spark.read.options(mode="PERMISSIVE", dropMalformed=True).csv("path/to/malformed_data.csv", header=True, inferSchema=True)
print(f"Number of rows read successfully: {df_permissive.count()}")

Handle Corrupted Records Using _corrupt_record Attribute: When using PERMISSIVE DROPMALFORMED, PySpark provides the _corrupt_record attribute which contains information about the corrupted or malformed records. This can be used to clean the data or log these issues.

#### Example
Code

Let’s see an example of how to use PERMISSIVE DROPMALFORMED and handle _corrupt_record.

PYTHON
from pyspark.sql import SparkSession

# Initialize Spark session
spark = SparkSession.builder.getOrCreate()

# Read CSV with PERMISSIVE DROPMALFORMED mode
df_permissive = spark.read.options(mode="PERMISSIVE", dropMalformed=True).csv("path/to/malformed_data.csv", header=True, inferSchema=True)

# Count the number of rows successfully read
print(f"Number of rows read: {df_permissive.count()}")

# Check for and print corrupted records
for row in df_permissive.select("_corrupt_record").collect():
    if row["_corrupt_record"] is not None:
        print(f"Corrupted record detected: {row['_corrupt_record']}")

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, FAILFAST mode halts processing on the first malformed record and throws an exception, while PERMISSIVE DROPMALFORMED allows data ingestion to continue but logs or marks corrupted records. The _corrupt_record attribute is a useful tool for identifying and handling these issues.

#### Common mistakes
include not using error handling mechanisms like try-except blocks when working with large datasets, failing to leverage the _corrupt_record attribute effectively, and misunderstanding the implications of FAILFAST vs PERMISSIVE DROPMALFORMED.

#### Follow-up
questions might explore more advanced techniques for cleaning or validating data, such as integrating validation rules into your data processing pipeline.

**Code:**

```python
from pyspark.sql import SparkSession

# Example code to handle corrupt records in PySpark
spark = SparkSession.builder.getOrCreate()

try:
    df_failfast = spark.read.csv("path/to/malformed_data.csv", header=True, inferSchema=True)
except Exception as e:
    print(f"FAILFAST mode encountered an error: {e}")

df_permissive = spark.read.options(mode="PERMISSIVE", dropMalformed=True).csv("path/to/malformed_data.csv", header=True, inferSchema=True)

for row in df_permissive.select("_corrupt_record").collect():
    if row["_corrupt_record"] is not None:
        print(f"Corrupted record detected: {row['_corrupt_record']}")
```

