---
type: "practice"
difficulty: "Intermediate"
---

# Why might you choose Parquet over CSV for a pipeline that feeds an analytics warehouse?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Pipeline Design Patterns](<../../../../02_Data_Engineering/Pipelines_and_Integration/Pipeline_Design/Pipeline_Design_Patterns.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a data pipeline that feeds into an analytics warehouse, one of the key considerations is the format in which data is stored and processed. CSV (Comma-Separated Values) files are simple and widely supported but come with limitations when it comes to handling large datasets and complex data structures. Parquet, on the other hand, offers several advantages that make it a better choice for such scenarios.

Parquet is designed specifically for analytics use cases and provides efficient storage and retrieval of structured data. It allows for columnar storage, which means that data can be stored in columns rather than rows, enabling faster querying and analysis over large datasets. Let's dive into the specifics to understand why Parquet might be preferred over CSV.

**Code:**

```python
# Example code snippet to illustrate reading from a CSV file vs Parquet file using Python with Pandas
import pandas as pd

csv_df = pd.read_csv('data.csv')
parquet_df = pd.read_parquet('data.parquet')
```

**Example input / output:**

```

Input:

PYTHON
Copy
# Example of reading CSV and Parquet files
csv_df = pd.read_csv('data.csv')
parquet_df = pd.read_parquet('data.parquet')

# Output - first few rows of each dataframe
print(csv_df.head())
print(parquet_df.head())

Output:

CODE
Copy
   column1  column2
0        a        x
1        b        y
2        c        z

     column1  column2
0         a        x
1         b        y
2         c        z
```

