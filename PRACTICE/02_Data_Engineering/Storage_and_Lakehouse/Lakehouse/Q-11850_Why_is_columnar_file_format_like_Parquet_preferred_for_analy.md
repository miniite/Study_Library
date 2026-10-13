---
type: "practice"
difficulty: "Basic"
---

# Why is columnar file format like Parquet preferred for analytical data in a lake?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Columnar Formats in the Lake](<../../../../02_Data_Engineering/Storage_and_Lakehouse/Lakehouse/Columnar_Formats_in_the_Lake.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Data Lake, we store vast amounts of raw and semi-processed data from various sources. Analyzing this data requires efficient retrieval and processing. Columnar file formats like Parquet are preferred because they offer significant performance benefits by optimizing storage and query execution.

The core idea is that columnar formats like Parquet store data in columns rather than rows, which allows for faster analytical queries. This format minimizes the amount of disk I/O needed when querying specific columns, reducing overall processing time.

**Comparison:**

```
Column	Description
name	Employee's Name
age	Employee's Age
salary	Employee's Salary
```

**Code:**

```python
# Example of reading a Parquet file using PyArrow (a library to handle columnar storage)
import pyarrow.parquet as pq

# Load data from Parquet file
table = pq.read_table('path/to/parquet/file.parquet')
print(table.to_pandas())  # Convert to pandas DataFrame for inspection
```

**Example input / output:**

```text
# Example input and output for reading a Parquet file using PyArrow
input = 'path/to/parquet/file.parquet'
output = pq.read_table(input).to_pandas()
print(output)
```

**Data:**

```json
{
  "example_data": [
    {"name": "Alice", "age": 28, "salary": 60000},
    {"name": "Bob", "age": 35, "salary": 75000},
    {"name": "Charlie", "age": 41, "salary": 90000}
  ]
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Query Specific Columns
6. Does the data store usecolumnar format?
7. Read full row
8. Are you querying specificcolumns?
9. Read all columns
10. Read only selected columnsand corresponding rows

