---
type: "practice"
difficulty: "Basic"
---

# Why does storing data by column let you read only the columns a query needs?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Row vs Columnar Formats](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Row_vs_Columnar_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When storing data in a columnar format like Parquet, ORC, or Avro, organizing the data by columns rather than rows significantly enhances the efficiency of queries that access specific columns. This is because columnar storage allows you to read only the necessary columns, which reduces both I/O operations and memory usage.

#### Triaging the Question

The core of this question revolves around understanding how columnar storage improves query performance by reading only the required data. We need to break down what "columnar" means in terms of file organization, explain why row-based systems might not be as efficient for partial queries, and provide a step-by-step explanation.

#### First-Principles Solution
Columnar Storage Basics: In columnar storage, each column is stored together rather than being intermingled with other columns. This means that all values of one column are grouped together.
Query Performance: When querying specific columns, you can read only the relevant part of the file, significantly reducing I/O operations compared to reading an entire row.
Efficiency and Storage Utilization: Columnar storage optimizes for queries that access a subset of columns by minimizing disk reads and cache misses.
Detailed Explanation

Let's consider a table with 10 columns and 1 million rows. If you need data from only two specific columns, a row-based system would still read the entire 10 columns for each of the 1 million rows. This results in unnecessary I/O operations and high memory usage.

Step-by-Step Breakdown
Reading Rows vs Columns: In a row-based system, you access the data by reading complete rows. For example, if your query involves two specific columns out of ten, you would still need to read all other columns for each row.
Columnar Storage Read Efficiency: With columnar storage, only the required columns are read. The query can directly access and process the relevant parts of the file, reducing I/O operations by a significant margin.

#### Example

#### Cons
ider a dataset with 100 rows and 5 columns:

Row	Column A	Column B	Column C	Column D	Column E
1	1	2	3	4	5
2	6	7	8	9	10

In a row-based system, reading rows 1 and 2 would require access to all columns. In contrast, with columnar storage, you could read only Column A and Column B for both rows.

#### Closing Summary

Storing data in a columnar format allows for more efficient query execution by reducing the amount of unnecessary I/O operations. Only the required columns are accessed during queries, which can significantly improve performance compared to row-based storage.

#### Common Mistakes/Follow-Ups
Misunderstanding Row-Based vs Column-Based: It's easy to confuse the two and not fully grasp why columnar storage is beneficial.
Ignoring Data Skew: While columnar storage is efficient for queries, it might not be optimal if there is significant data skew across columns.
Performance in Aggregation Queries: In scenarios where complex aggregation operations are required on multiple columns, the benefits of columnar storage may diminish.

Would you like to delve deeper into any specific aspect or explore other use cases?

**Comparison:**

| Row | ColumnA | ColumnB |
|-----|---------|---------|
| 1   | 1       | 2       |
| 2   | 6       | 7       |

**Code:**

```python
# Example code for reading a Parquet file with only necessary columns using PyArrow
import pyarrow.parquet as pq

# Load the dataset
dataset = pq.ParquetDataset('path/to/dataset.parquet')

# Select only the required columns
required_columns = ['ColumnA', 'ColumnB']
selected_dataset = dataset.read(columns=required_columns)

print(selected_dataset)
```

**Example input / output:**

```text
Input: Read a Parquet file with only 'ColumnA' and 'ColumnB'
Output:
[{
  "RowID": 1,
  "ColumnA": 1,
  "ColumnB": 2
}, {
  "RowID": 2,
  "ColumnA": 6,
  "ColumnB": 7
}]
```

**Data:**

```json
{
  "data": [
    {
      "RowID": 1,
      "ColumnA": 1,
      "ColumnB": 2,
      "ColumnC": 3,
      "ColumnD": 4,
      "ColumnE": 5
    },
    {
      "RowID": 2,
      "ColumnA": 6,
      "ColumnB": 7,
      "ColumnC": 8,
      "ColumnD": 9,
      "ColumnE": 10
    }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Read Full Row
4. Are Columns Required?
5. Select and Read OnlyNecessary Columns
6. Do Nothing
7. Process Data Efficiently

