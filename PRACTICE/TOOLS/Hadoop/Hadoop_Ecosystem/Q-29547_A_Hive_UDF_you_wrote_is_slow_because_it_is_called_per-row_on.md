---
type: "practice"
difficulty: "Deep"
---

# A Hive UDF you wrote is slow because it is called per-row on billions of rows. How would you optimise or replace it with a native/built-in approach?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hive Partitioning and Bucketing](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/Hive_Partitioning_and_Bucketing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To optimize a Hive User Defined Function (UDF) that processes each row individually in a large dataset, we need to rethink how the operation can be performed more efficiently using built-in functions or by modifying our data processing approach. The key is to reduce the number of function calls and leverage vectorized operations which are inherently faster due to optimized internal implementation.

The UDF was likely written in Java, Python, or another language for Hive, but it's being called per-row, leading to a slow execution time when applied to billions of rows. To address this, we can explore several strategies:

Aggregate Data Before Applying the Function: Instead of applying the function to each row individually, aggregate data and then apply the UDF on the aggregated results.
Use Vectorized UDAF (User Defined Aggregation Functions): Hive supports vectorized aggregation functions that process multiple rows at once, significantly reducing the overhead.
Optimize Data Formats: Use more efficient storage formats like Parquet or ORC that support columnar storage and allow for faster processing.
MapReduce vs Spark: Consider using Apache Spark if you can rewrite your UDF in a Spark context, which supports distributed computing.

Let's break down each approach:

#### Triage

The question is asking us to optimize an existing Hive UDF by either:

Modifying the function itself (using built-in functions or vectorization).
Transforming the data processing pipeline.
Using a different framework that better suits the task, like Apache Spark.

We will assume the current UDF has some specific logic that needs optimization. For example, let's consider a simple case where we need to calculate the average of a column after applying some transformation on it:

SQL
CREATE FUNCTION my_udf AS 'com.example.MyUDF';

In our original code, this function is being used like so:

SQL
SELECT my_udf(column) FROM table;

#### First-Principles Solution

Aggregate Data Before Applying the Function:

Instead of applying my_udf per row, we can first aggregate data and then apply it.
SQL
SELECT AVG(my_udf(column)) FROM table;

Use Vectorized UDAF:

Hive supports built-in vectorized functions that operate on multiple rows at once.

For example, if our transformation is a simple arithmetic operation, we can use the built-in AVG function directly:

SQL
SELECT AVG(column * 2) FROM table;

Optimize Data Formats:

Use columnar storage formats like Parquet or ORC to improve read performance.

MapReduce vs Spark:

If our UDF is complex and can be rewritten in a distributed computing framework, we might consider using Apache Spark.
Detailed Steps

Aggregate Data Before Applying the Function:

Modify the SQL query to first aggregate data before applying my_udf.

Use Vectorized UDAF:

Check if Hive's built-in functions can replace our custom UDF.
For example, use AVG or other vectorized functions that process multiple rows.

Optimize Data Formats:

Convert the data to a more efficient format like Parquet or ORC.

MapReduce vs Spark:

If none of the above strategies work, consider rewriting our logic in Spark.

#### Closing Summary

By leveraging built-in functions, optimizing data formats, and possibly using Apache Spark, we can significantly reduce the execution time of our Hive UDF. The key is to minimize the number of function calls and take advantage of vectorized operations.

#### Common Mistakes/Follow-Ups
Not Understanding Built-In Functions: Many users write custom UDFs when built-in functions are available.
Ignoring Data Formats: Using inefficient storage formats can slow down queries even if the logic is optimized.
Overlooking Distributed Computing Frameworks: For complex operations, Spark might provide better performance than Hive.

**Comparison:**

  | id | value       |
  | --:| :---------- |
  | 1  | 'abc'       |
  | 2  | 'def'       |
  | 3  | 'ghi'       |

**Code:**

```

// Original UDF example in Java public class MyUDF extends UDF { public String evaluate(String input) { // Some transformation return "transformed_" + input; } }

CODE
Copy

```sql
CREATE FUNCTION my_udf AS 'com.example.MyUDF';
```

**Example input / output:**

```

Input:

SQL
Copy
SELECT my_udf(value) FROM example_table;

Output:

SQL
Copy
+------------------------------------------------+
| transformed_value1 |
| transformed_value2 |
| ...                |
+------------------------------------------------+
```

**Data:**

```json
{
  "table": {
    "name": "example_table",
    "columns": [
      {"name": "id", "type": "int"},
      {"name": "value", "type": "string"}
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Read Data
4. Is UDF Complex?
5. Convert to Spark
6. Aggregate Data BeforeApplying UDF
7. Use Vectorized UDAF
8. Optimize Storage Format

