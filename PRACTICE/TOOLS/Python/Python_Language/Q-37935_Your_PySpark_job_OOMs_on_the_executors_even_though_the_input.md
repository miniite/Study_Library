---
type: "practice"
difficulty: "Deep"
---

# Your PySpark job OOMs on the executors even though the input is small. What driver-vs-executor memory mistakes (collect, broadcast, wide UDFs) would you check first?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Memory Leaks in Batch Jobs](<../../../../TOOLS/Python/Python_Language/Memory_Leaks_in_Batch_Jobs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When encountering Out-of-Memory (OOM) errors in a PySpark job despite the input being relatively small, it’s crucial to diagnose whether the problem lies with the driver or the executors. The common suspects often involve memory management issues such as collect, broadcast variables, and wide User-Defined Functions (UDFs). Here's how we would approach this:

Firstly, OOM errors in PySpark typically indicate that your application is consuming more memory than available on either the driver or executor nodes. Given a small input dataset, it’s likely not the data size itself causing the issue but rather inefficient use of resources.

#### Triage
What Is Being Probed

The interviewer wants to understand how you think about memory management in PySpark and identify common pitfalls that can lead to OOM errors. Specifically, they want to see if you recognize potential issues with:

Collecting data on the driver
Broadcasting large datasets or UDFs
Performing operations that require extensive memory usage

#### Assumptions
The dataset is small enough for local execution but still triggers an OOM error.
The issue is related to inefficient resource utilization rather than a truly large input.
Step-by-Step Solution from First Principles
1. Check Executor Memory Usage

Start by checking the memory configuration of the executors:

PYTHON
# Assuming Spark session or context named spark
spark.sparkContext.conf.get("spark.executor.memory")

Ensure that each executor has sufficient memory allocated for tasks. If not, adjust the --executor-memory flag when starting the Spark job.

2. Review UDFs and Narrow vs Wide Functions

User-Defined Functions (UDFs) can be a significant source of memory issues if they process large amounts of data. Identify any wide functions that might be causing problems:

PYTHON
# Example: A UDF that aggregates data across partitions
from pyspark.sql.functions import udf
from pyspark.sql.types import IntegerType

@udf(returnType=IntegerType())
def aggregate_func(data):
    return sum(data)

# Check for wide transformations
spark.udf.getAllGlobalFunctions()

Narrow functions, which operate within a single partition and do not require data shuffling between partitions, are generally more efficient. Wide operations that shuffle large amounts of data can cause memory issues.

3. Investigate Broadcast Variables

Broadcast variables are used to share small datasets across multiple tasks efficiently. However, if improperly used, they can lead to OOM errors:

PYTHON
# Example: Creating and using a broadcast variable
from pyspark import SparkContext

sc = SparkContext.getOrCreate()
broadcast_var = sc.broadcast([1, 2, 3])

Check your code for large datasets being broadcasted. If you find that a broadcast is holding a significant amount of data, consider if it’s necessary or if there's another way to achieve the same result.

4. Review Data Collection on Driver

Collecting data on the driver can be memory-intensive and should be used judiciously:

PYTHON
# Example: Collecting an RDD on the driver
rdd = spark.sparkContext.parallelize([1, 2, 3])
data = rdd.collect()

If you are using collect to gather results from an executor, ensure that the amount of data being collected is manageable. Consider alternative approaches such as writing data to a file or partitioning the dataset more effectively.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, when faced with OOM errors in PySpark despite small input sizes, you should focus on memory management practices like executor settings, UDF optimization, broadcast variables, and driver-side operations. The key is to identify inefficient use of resources and refactor your code accordingly.

#### Common mistakes
include:

Overusing collect to gather data from executors.
Improperly using broadcast variables for large datasets.
Failing to optimize wide transformations like groupBy, reduceByKey, etc., which can consume significant memory.

#### Follow-ups
might involve diving deeper into specific code snippets, discussing the trade-offs between different optimization techniques, or exploring alternative frameworks or approaches that could help mitigate these issues.

