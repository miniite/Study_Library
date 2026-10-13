---
type: "practice"
difficulty: "Basic"
---

# Why do columnar formats compress better than row formats?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Row vs Columnar Formats](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Row_vs_Columnar_Formats.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To answer why columnar formats like Parquet, ORC, and Avro compress better than row formats, we need to understand their underlying data storage structures and compression techniques. Let's break it down from first principles.

Firstly, row formats store data in a sequential manner where each row contains all the columns' values end-to-end. In contrast, columnar formats store data by column groups, which means that all values of one column are stored together. This structural difference is crucial for compression because it allows columnar storage to take advantage of inherent patterns and redundancies within each column.

Now, let's dive into the details step-by-step:

Data Patterns in Columnar Formats:

When data is grouped by columns, commonalities across rows are reduced. For example, a date column might have many repeated values, which can be efficiently compressed.

Run-Length Encoding (RLE):

RLE is a simple and effective compression technique where sequences of identical data elements are stored as a single data value and count. In columnar formats, this works exceptionally well because columns often contain repetitive patterns.

Dictionary Encoding:

Dictionary encoding maps unique values to integers, reducing the number of bytes required to represent them. This is particularly useful in columns with many distinct but infrequently repeated values.

Data Types and Precision:

Columnar formats can take advantage of data types (e.g., integer vs. string) to apply more efficient encoding techniques like bit packing or delta encoding, which further compress the data.

Sliced Compression:

Since columns are stored separately in columnar formats, compression algorithms can be applied independently and in parallel across different columns, potentially improving overall compression ratios.

Performance Trade-offs:

While columnar formats offer better compression, they generally require more memory and processing power during reads due to the need to seek through multiple segments of data for each query.

In summary, the core reason why columnar formats compress better is their ability to leverage patterns within columns, making them ideal for scenarios where read performance and storage efficiency are both important. However, the trade-offs must be considered in practice.

**Comparison:**

| Compression Technique | Description                                  |
|-----------------------|----------------------------------------------|
| Run-Length Encoding   | Stores sequences of identical data elements  |
| Dictionary Encoding   | Maps unique values to integers               |
| Bit Packing           | Reduces integer sizes                        |
| Delta Encoding        | Saves changes relative to previous value     |

**Code:**

```text
# Example Python code using Pandas to demonstrate difference between row vs column compression
import pandas as pd

# Generate sample data
data = {'id': [1, 2, 3] * 100,
        'name': ['Alice', 'Bob', 'Charlie'] * 100,
        'age': [25, 30, 35] * 100}
df_row = pd.DataFrame(data)
df_col = df_row.copy().T

# Convert to Parquet (emulated for simplicity)
parquet_row = df_row.to_parquet('row.parquet')
parquet_col = df_col.to_parquet('col.parquet')

import os
print(f"Row format size: {os.path.getsize('row.parquet') / 1024:.2f} KB")
print(f"Columnar format size: {os.path.getsize('col.parquet') / 1024:.2f} KB")
```

**Example input / output:**

```

Input (Row format):

CODE
Copy
import pandas as pd
df = pd.read_parquet('row.parquet')
print(df)

Output:

CODE
Copy
   id      name  age
0    1     Alice   25
1    2       Bob   30
2    3  Charlie   35
3    1     Alice   25
4    2       Bob   30
...

Input (Columnar format):

CODE
Copy
import pandas as pd
df = pd.read_parquet('col.parquet')
print(df)

Output:

CODE
Copy
      id         name          age
0     1.0        Alice        25.0
98    3.0      Charlie        35.0
...
```

**Data:**

```json
{
    "id": [1, 2, 3],
    "name": ["Alice", "Bob", "Charlie"],
    "age": [25, 30, 35]
}
```

**Flow:**

1. Columnar Storage
2. RLE Compression
3. Dictionary Encoding
4. Bit Packing
5. Delta Encoding

