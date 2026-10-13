---
type: "practice"
difficulty: "Basic"
---

# What are the three main open table formats used in lakehouses and can you name them?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Columnar Formats in the Lake](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Columnar_Formats_in_the_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

In a data lake or lakehouse, "open table formats" refer to file formats that support schema evolution, making it easier to manage evolving data over time. The three main open table formats used in lakehouses are Parquet, ORC (Optimized Row Columnar), and Delta Lake. This answer covers their key characteristics and why they are chosen.

#### Triage

The question is asking about the most commonly used open table formats for storing data in a lakehouse architecture. These formats need to support schema evolution to accommodate changes over time. The interviewer might be probing your understanding of different file formats, their advantages, and how they fit into modern data storage strategies.

#### First-Principles Solution

There are three primary open table formats used in lakehouses: Parquet, ORC (Optimized Row Columnar), and Delta Lake. Each format has its strengths and is chosen based on specific requirements such as performance, schema evolution support, and data management capabilities.

Parquet

Key Characteristics: Parquet is a columnar storage file format that separates data into columns instead of rows, which can lead to more efficient queries by reading only the necessary parts.
Use Cases: Suitable for scenarios where large datasets need to be stored and queried efficiently.

ORC (Optimized Row Columnar)

Key Characteristics: ORC is a file format designed for Hadoop that offers better query performance through various optimizations like dictionary encoding, striping, and predicate pushdown.
Use Cases: Useful when you need to process large datasets with complex queries and require efficient storage and retrieval.

Delta Lake

Key Characteristics: Delta Lake is built on top of existing file formats like Parquet but provides ACID transactions, schema enforcement, and version control.
Use Cases: Ideal for environments requiring robust data management features such as transactional semantics, lineage tracking, and metadata operations.

#### Closing Summary

In summary, the three main open table formats used in lakehouses are Parquet, ORC, and Delta Lake. Each has its unique strengths and is selected based on specific requirements of the use case. Understanding these formats helps in designing efficient data storage solutions that can evolve over time while ensuring robustness and performance.

#### Common Mistakes/Follow-Ups

Mistake: Confusing Parquet with ORC, or vice versa.

#### Follow-Up
Clarify the differences by explaining the optimizations of ORC and the columnar nature of Parquet.

Mistake: Not mentioning Delta Lake as a modern solution for transactional data lakes.

#### Follow-Up
Explain how Delta Lake brings ACID transactions to big data environments, making it suitable for real-time applications.

#### Example

Here is an example input and output:

PLAINTEXT
Input: Parquet vs ORC vs Delta Lake

Output:
- Parquet is columnar storage optimized for large datasets.
- ORC supports dictionary encoding and predicate pushdown for efficient queries.
- Delta Lake provides ACID transactions on top of existing formats.

**Comparison:**

| Format | Key Characteristics                         | Use Cases                                                                 |
|--------|---------------------------------------------|-------------------------------------------------------------------------|
| Parquet | Columnar storage, optimized for large datasets | Large-scale data processing and analytics                             |
| ORC     | Supports dictionary encoding & predicate pushdown | Complex queries with efficient storage                                  |
| Delta Lake  | ACID transactions, schema enforcement, version control | Robust transactional semantics, lineage tracking, metadata operations   |

**Code:**

```python
# Example Python code to read data using these formats (hypothetical)
from pyspark.sql import SparkSession

spark = SparkSession.builder.appName("Example").getOrCreate()

# Reading Parquet
parquet_df = spark.read.parquet("path/to/parquet/directory")

# Reading ORC
orc_df = spark.read.format("orc").load("path/to/orc/directory")

# Reading Delta Lake
delta_df = spark.read.format("delta").load("path/to/delta/lake")
```

**Example input / output:**

```text
Input: Parquet vs ORC vs Delta Lake

Output:
- Parquet is columnar storage optimized for large datasets.
- ORC supports dictionary encoding and predicate pushdown for efficient queries.
- Delta Lake provides ACID transactions on top of existing formats.
```

**Data:**

```json
{
  "parquet": {
    "fileFormat": "Parquet",
    "columns": ["id", "name", "age"],
    "schemaEvolution": true
  },
  "orc": {
    "fileFormat": "ORC",
    "columns": ["id", "name", "age"],
    "schemaEvolution": true,
    "optimizations": ["dictionary encoding", "striping"]
  },
  "delta": {
    "fileFormat": "Delta Lake",
    "columns": ["id", "name", "age"],
    "features": ["ACID Transactions", "Schema Enforcement", "Version Control"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Parquet
6. Is your dataset large?
7. Choose Parquet
8. Consider ORC or Delta Lake
9. Do you need ACIDtransactions?
10. Choose Delta Lake
11. Choose ORC for optimizations

