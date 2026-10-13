---
type: "practice"
difficulty: "Intermediate"
---

# Why is calling .collect() on a large PySpark DataFrame dangerous, and what alternatives (toLocalIterator, write, foreachPartition) would you use to move data out safely?

**Difficulty:** Intermediate · **Levels:** SDE-1/2, SDE-2/3 · [↑ Pandas vs Polars vs PySpark](<../../../../TOOLS/Python/Python_Language/Pandas_vs_Polars_vs_PySpark.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When working with PySpark DataFrames that are large in size, calling .collect() can pose significant risks due to its inherent behavior of loading all the data into the driver's memory. This approach can lead to out-of-memory errors or performance bottlenecks if not handled carefully. In this response, I'll explain why .collect() is dangerous and explore safer alternatives such as toLocalIterator, writing to files, and using foreachPartition.

**Code:**

```python
# Example of .collect() usage (potentially dangerous)
large_df = spark.read.parquet("path/to/large/data")
result = large_df.collect()
```

