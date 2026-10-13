---
type: "practice"
difficulty: "Deep"
---

# What are repetition and definition levels in Parquet and roughly what do they encode?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Parquet Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Parquet_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Repetition and definition levels are key components of the data layout in Apache Parquet, a columnar storage format that optimizes reading large datasets efficiently. At a high level, repetition and definition levels help reduce the overall file size by encoding metadata about the structure of the data, allowing for more efficient querying.

To delve deeper, we need to understand what these terms mean:

Definition Levels: These encode whether a row group contains an entry or not.
Repetition Levels: These indicate how many times each value in a column is repeated within a chunk of rows.

In essence, repetition and definition levels help Parquet manage data efficiently by encoding metadata at different granularities. This approach significantly reduces the storage footprint for sparse datasets and optimizes query performance by allowing Parquet readers to skip irrelevant parts of the file.

To illustrate with an example:

MARKDOWN
| RepLevel | DefLevel |
|----------|----------|
| 0        | 0        | No repetition, no definition (null value) |
| 1        | 0        | No repetition, defined (non-null value)   |
| 1        | 1        | Repeated once, defined                   |

Let's break down the concepts in more detail:

#### Triage

The question is probing your understanding of Parquet's internal data structure and its optimization techniques. The assumptions here are that you should be familiar with basic Parquet concepts such as columnar storage, row groups, and compression.

#### First-Principles Solution

Parquet uses a hierarchical encoding scheme for its metadata to optimize storage and query performance. Each value in the file is encoded at two levels: repetition level (RepLevel) and definition level (DefLevel).

Repetition Level:

The repetition level indicates how many times each value is repeated within a chunk of rows. For instance, if you have a column with 10 rows, but only the first row has a non-null value, the other nine will be nulls. In this case, the repetition level for these nine null values would be 2 (since they are repeated from the second row onwards).

Definition Level:

The definition level indicates whether each row in the chunk contains an entry or not. If all rows have the same non-null value, the definition level is 0 because there's no need to encode which rows contain data and which do not.

#### Closing Summary

In summary, repetition and definition levels are critical for optimizing Parquet storage by encoding metadata about how values are repeated and defined within a chunk of rows. This helps in reducing the overall file size and improves query performance by allowing efficient skipping of irrelevant parts of the file.

#### Common Mistakes/Follow-Ups
Misunderstanding: A common mistake is confusing repetition and definition levels with each other. Ensure you understand that RepLevel concerns how values are repeated, while DefLevel indicates which rows contain data.
Further Exploration: Interviewers might ask about how these levels interact with compression techniques or how they impact query performance in specific scenarios.

**Comparison:**

| Row | Value   | RepLevel | DefLevel |
|-----|---------|----------|----------|
| 1   | None    | 0        | 0        |
| 2   | SomeVal | 1        | 1        |
| 3   | SomeVal | 1        | 1        |
| 4   | None    | 2        | 0        |

**Code:**

```python
# Example of Parquet file structure using Apache Arrow (Python)
import pyarrow.parquet as pq

# Load a Parquet file
file = pq.ParquetFile('path/to/file.parquet')

# Get metadata information, including repetition and definition levels
metadata = file.metadata

print(metadata.repetition_level)  # RepLevel info
print(metadata.definition_level)  # DefLevel info
```

**Example input / output:**

```text
Input:
- A Parquet file with a column 'values' that has several nulls and non-null values

Output:
- RepLevel for the chunk where repetition starts from the first row: [0, 1, 2]
- DefLevel indicating which rows have defined data: [0, 1]

Explanation:
- The first row (value = None) has a RepLevel of 0 because it's not repeated.
- The second row (value = SomeValue) has a RepLevel of 1 because its value is repeated twice in the chunk.
- The third row (value = SomeOtherValue) has a RepLevel of 2, indicating it’s only defined once and then repeats.
```

**Data:**

```json
{
    "repetitionLevels": [0, 1, 2],
    "definitionLevels": [0, 1]
}
```

**Flow:**

1. Yes
2. No
3. Data Present
4. Data Absent
5. Define RepLevel
6. Value Repeated?
7. Increment RepLevel, RepeatValue
8. Reset RepLevel to 0, SetDefLevel for new entry
9. Check if row has data(DefLevel)
10. Set DefLevel = 1
11. Set DefLevel = 0

