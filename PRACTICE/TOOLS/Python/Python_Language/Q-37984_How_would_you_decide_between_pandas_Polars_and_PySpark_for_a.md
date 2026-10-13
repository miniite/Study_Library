---
type: "practice"
difficulty: "Deep"
---

# How would you decide between pandas, Polars, and PySpark for a transform, and what makes Polars' lazy engine and Arrow backing faster than pandas on a single machine?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pandas vs Polars vs PySpark](<../../../../TOOLS/Python/Python_Language/Pandas_vs_Polars_vs_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between pandas, Polars, and PySpark for data transformation tasks in Python, you need to consider several factors including performance, memory efficiency, parallelism capabilities, ease of use, and specific requirements of the project. Here's a deep dive into why Polars might be faster than pandas on a single machine due to its lazy evaluation engine and Arrow backing.

Firstly, let’s triage the question:

Purpose: You are asked to compare pandas, Polars, and PySpark.
Key Aspects: The focus is specifically on why Polars is faster than pandas when running on a single machine.

#### Assumptions
We assume you have basic knowledge of these libraries.

Now, let's break down the solution step by step:

Step 1: Understanding pandas

Pandas is a powerful library for data manipulation and analysis in Python. It’s based on NumPy and provides easy-to-use data structures and operations for manipulating numerical tables and time series. However, its synchronous and eager evaluation nature means that it processes all the data before producing any results.

Step 2: Understanding Polars

Polars is a modern library built specifically to handle large datasets efficiently. It leverages the Rust language, which allows for very fast execution speeds due to optimizations like zero-cost abstractions, just-in-time compilation (JIT), and efficient memory management. One of its key features is the lazy evaluation engine.

Step 3: Lazy Evaluation

Lazy evaluation in Polars means that operations are not executed immediately but only when a terminal operation (like .collect()) is called. This allows for better optimization, as multiple data manipulations can be combined into a single computation plan before any actual processing occurs. For example:

PYTHON
import polars as pl

df = pl.scan_csv("data.csv")  # Lazy DataFrame creation
result = (
    df.filter(pl.col("col1") > 10)
    .select(["col2", "col3"])
    .groupby("category")
    .agg([pl.sum("col4"), pl.mean("col5")])
).collect()  # Terminal operation triggers execution
Step 4: Arrow Backend

Polars uses the Apache Arrow format for data exchange and internal storage. Arrow is designed to be efficient across languages, allowing Polars to handle large datasets more efficiently than other libraries that use in-memory Python objects.

Step 5: Memory Efficiency

Polars is optimized to work with larger datasets by minimizing memory overhead. For instance, it supports data types like Int8, which can significantly reduce the memory footprint for columns with a wide range of values.

Step 6: Performance Comparison

In many scenarios, especially when dealing with large datasets or complex transformations, Polars outperforms pandas. A simple comparison can be seen in operations such as filtering, grouping, and aggregation:

Pandas:

PYTHON
df = pd.read_csv("data.csv")
result = df[df['col1'] > 10]
result = result.groupby('category').agg({'col2': 'sum', 'col3': 'mean'})

Polars:

PYTHON
df = pl.scan_csv("data.csv")  # Lazy DataFrame creation
result = (
    df.filter(pl.col("col1") > 10)
    .select(["col2", "col3"])
    .groupby("category")
    .agg([pl.sum("col4"), pl.mean("col5")])
).collect()  # Terminal operation triggers execution
Step 7: Example Scenario

#### Cons
ider a scenario where you need to filter, group by category, and perform multiple aggregations on a large dataset. Polars can handle this more efficiently due to its lazy evaluation:

PYTHON
# Sample Polars code
df = pl.scan_csv("large_dataset.csv")
result = (
    df.filter(pl.col("value") > 10)
    .select(["category", "value"])
    .groupby("category")
    .agg([pl.sum("value"), pl.mean("value")])
).collect()

#### Closing Summary

Polars is faster than pandas on a single machine due to its lazy evaluation engine and efficient use of the Arrow format. This approach allows for better optimization and reduced memory overhead, making it ideal for large-scale data processing tasks.

#### Common Mistakes/Follow-Ups
Misunderstanding Lazy Evaluation: Failing to recognize that operations are deferred until explicitly triggered can lead to inefficient code.
Not Leveraging Arrow Efficiently: Ignoring the benefits of using Arrow format for memory efficiency and cross-language compatibility.
Overlooking PySpark’s Parallelism: Assuming Polars is always better when parallel processing or distributed computing might be required.

**Comparison:**

```
category	sum(col4)	mean(col5)
A	45	65.0
B	37	60.0
```

**Code:**

```python
import polars as pl

df = pl.scan_csv("data.csv")
result = (
    df.filter(pl.col("col1") > 10)
    .select(["col2", "col3"])
    .groupby("category")
    .agg([pl.sum("col4"), pl.mean("col5")])
).collect()
```

**Example input / output:**

```text
Input:
data.csv

Output:
category  sum(col4)  mean(col5)
A         45         65.0
B         37         60.0
```

**Data:**

```
CSV
Copy
data.csv
category,col1,col2,col3,col4,col5
A,10,30,40,20,60
A,15,45,55,25,70
B,8,22,33,18,58
B,9,27,39,19,61
```

**Flow:**

1. Yes
2. Yes
3. No
4. Start
5. Read CSV with Polars
6. Is Data Filtered?
7. Lazy Filter
8. Lazy Select
9. Lazy GroupBy
10. Are Aggregations Needed?
11. Lazy Aggregation
12. Collect Results
13. End

