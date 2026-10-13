---
type: "practice"
difficulty: "Basic"
---

# What is the difference between structured semi-structured and unstructured data and how does Hadoop handle each?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ HDFS Fundamentals](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/HDFS_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a nutshell, structured data has a fixed schema that fits neatly into rows and columns, like in traditional relational databases. Semi-structured data has some form of structure but is more flexible than structured data; it includes elements like JSON or XML documents with tags and attributes. Unstructured data lacks any predefined schema and can include things like images, videos, text files, etc. Hadoop handles each type differently: it uses MapReduce to process large volumes of semi-structured data effectively, while its distributed file system (HDFS) is well-suited for storing unstructured data.

To delve deeper, let's break down how Hadoop processes and stores these types of data.

**Code:**

```
Example code snippet to illustrate reading structured data in Hadoop using PySpark

from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Example").getOrCreate() df_structured = spark.read.csv("/path/to/structured_data", header=True, inferSchema=True)

CODE
Copy

```python
# Example for semi-structured JSON data
df_semi_structured = spark.read.json("/path/to/json_file")
```

**Flow:**

1. Yes
2. No
3. Structured Data
4. Is schema fixed?
5. Relational Databases
6. Semi-Structured Data
7. Hadoop MapReduce forprocessing, HDFS for storage
8. Unstructured Data
9. HDFS for storage andprocessing with tools like Hiveor Spark

