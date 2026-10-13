---
type: "practice"
difficulty: "Deep"
---

# Why can column statistics be misleading when a column contains many NULLs, and how does that affect row-group pruning decisions?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Row vs Columnar Formats](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Row_vs_Columnar_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Columnar file formats like Parquet, ORC, and Avro are designed to optimize data storage and retrieval by organizing data into columns rather than rows. However, when a column contains many NULL values, the utility of storing and using column statistics for pruning can be significantly diminished. This is because column statistics rely heavily on non-NULL values to make efficient decisions about which row groups to scan during query execution.

#### Triage
What's Being Probed

The question probes your understanding of how columnar storage works, particularly focusing on the impact of NULLs on column statistics and their practical use in data processing. The interviewer is likely interested in your ability to explain the limitations of these statistics and the implications for query optimization.

Key Assumptions
Understanding of basic principles of columnar storage.
Familiarity with concepts like column statistics, null values, and row-group pruning.
Knowledge of how databases use these statistics during query execution.

#### First Principles Solution
Column Statistics Overview

Column statistics in columnar file formats are metadata that capture information about the contents of a column. This includes:

Min/Max Values: The minimum and maximum values in the column.
Null Count: The number of null or missing values.
Cardinality: The number of distinct values.
Histograms: Distributions of value frequencies.
Impact of Many NULLs

When a column has many NULLs, the utility of these statistics diminishes because:

Min/Max Values: If a column contains many NULLs, the min and max values might not accurately represent the actual range of non-null data. For example, if most entries are NULL, the min could be an arbitrary small value, and the max could be an abnormally large one.
Null Count: The null count can skew the cardinality and other statistics. If a column has 80% NULLs, the effective range of non-null values is much smaller than suggested by overall stats.
Cardinality: High NULL counts reduce the effectiveness of cardinality estimates. This can lead to inefficient execution plans because the system might incorrectly assume there are many distinct non-null values.
Row-Group Pruning

Row-group pruning is a technique where the database engine decides which row groups (blocks) in a columnar store need to be read based on query conditions. If a column has many NULLs, this decision-making process can become unreliable:

Incorrect Group Selection: If statistics suggest a wide range of values but most are NULL, the system might select unnecessary row groups.
Increased I/O: Unnecessary scans of NULL-filled blocks lead to increased read operations and longer query execution times.
Practical Example

#### Cons
ider a table with a column age where 80% of entries are NULL:

Min/Max statistics: Might show min=1, max=99 (due to few non-null values).
Null count: Indicates 80% nulls.
Cardinality: Overestimates the number of distinct ages.

If a query filters on age > 25, the system might incorrectly assume many rows match this condition and read unnecessary row groups, leading to suboptimal performance.

#### Closing Summary

Column statistics are powerful tools for optimizing data retrieval in columnar storage. However, columns with high NULL counts can render these statistics less effective. This is because they misrepresent the true distribution of non-null values, leading to inefficient query execution plans and increased I/O operations.

#### Common Mistakes/Follow-Ups
Misinterpreting Null Counts: Failing to account for the impact of NULLs on overall statistics.
Ignoring Edge Cases: Overlooking scenarios where high NULL counts significantly affect columnar storage efficiency.
Lack of Contextual Understanding: Not considering how these statistics interact with other query optimization techniques.

#### Follow-ups
might include:

How to handle columns with many NULLs in practice?
What alternative strategies can be employed for better performance?

**Comparison:**

| Row Group | Min Age | Max Age | Null Count |
|-----------|---------|---------|------------|
| 1         | 2       | 98      | 450        |
| 2         | 36      | 74      | 100        |

**Code:**

```python
# Example code snippet to illustrate querying a columnar store
import pyarrow.parquet as pq

# Load Parquet file
table = pq.read_table('data.parquet')
age_column = table['age']

# Query condition
filtered_rows = age_column.filter(age_column > 25)

print(filtered_rows)
```

**Example input / output:**

```text
Input:
- Query: `SELECT * FROM table WHERE age > 25`

Output:
- Rows with age > 25: `[30, 45, 60]`

Dry-run shows that despite the query condition, many row groups might be unnecessarily read due to misleading column statistics.
```

**Data:**

```json
{
    "age": [null, null, 18, 30, 45, null, 60]
}
```

**Flow:**

1. Yes
2. No
3. Query Execution
4. Column Statistics Available?
5. Prune Row Groups
6. Full Table Scan Required
7. Read Selected Row Groups
8. Perform Full Scan

