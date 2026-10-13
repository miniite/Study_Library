---
type: "practice"
difficulty: "Deep"
---

# Explain how bitmap indexes compress and store row references, and why that makes them memory-efficient for low-cardinality columns.

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Bitmap Indexes](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Bitmap_Indexes.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

Bitmap indexes are a type of index used in databases to efficiently store and query low-cardinality data. They compress the storage by representing each column value as a bitmap (a binary string), which significantly reduces memory usage compared to storing the full row references.

#### Triage

The question is asking for an explanation of how bitmap indexes work, specifically focusing on their compression mechanism and why this makes them efficient in terms of memory usage for low-cardinality columns. The key aspects we need to cover include:

What a bitmap index is.
How it represents data using bitmaps.
Why this representation is memory-efficient.
Practical scenarios where bitmap indexes are useful.

#### First Principles Solution
Understanding Bitmap Indexes

Bitmap indexes are a type of index used in databases, particularly for low-cardinality columns. Low-cardinality means that the number of distinct values in the column is relatively small compared to the total number of rows. For example, gender (male/female) or status (active/inactive) are typical examples.

How Bitmap Indexes Work

In a bitmap index, each row's value in the low-cardinality column is represented as a bit in a binary string. Each bit corresponds to whether a particular row has that value. Consider a simple example with 10 rows and a gender column:

Row 1: Male -> Bit 0 (0th position)
Row 2: Female -> Bit 1 (1st position)
Row 3: Male -> Bit 0 (0th position again)
Row 4: Female -> Bit 1 (1st position again)
... and so on.

This results in a bitmap like 01010... for the column. Each bit corresponds to one of the possible values, with the position representing the row number.

Memory Efficiency

The key advantage of bitmap indexes is their memory efficiency. Instead of storing pointers or full row references, which can be large, we store compact bitmaps. For a low-cardinality column, the number of bits required per row is much smaller than the size of a pointer. This means that for each value in the column, you only need one byte to represent its presence in all rows where it appears.

#### Example

Let's assume we have 1000 rows and 5 possible gender values (Male, Female, Trans, Unknown, Other). Each bit corresponds to one of these five values. The bitmap would be a sequence of 2000 bits (400 bytes), which is much smaller than storing the actual row references.

Practical Scenarios

Bitmap indexes are particularly useful in scenarios where:

The column has a small number of distinct values.
You frequently query for rows based on these columns.
The table is large, and memory usage is a concern.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, bitmap indexes compress the storage by representing low-cardinality column values as bitmaps. This makes them highly efficient in terms of memory usage because they require much less space compared to storing full row references. However, they are not suitable for high-cardinality columns where there are many unique values.

#### Common mistakes
include:

Thinking that bitmap indexes work well for all types of data. They are primarily effective for low-cardinality columns.
Ignoring the fact that while memory efficient, bitmap indexes can be slower to scan than other index types due to their structure.

#### Follow-ups
could include discussing how to choose between different types of indexes (e.g., covering indexes, composite indexes) or how to optimize query performance using these indexes in specific use cases.

**Comparison:**

```
Row ID	Gender
1	Male
2	Female
3	Male
4	Female
...	...

Bitmap for 'Male': 0101...
```

**Code:**

```sql
-- Example SQL for creating a bitmap index
CREATE BITMAP INDEX idx_gender ON table_name(gender);
```

**Example input / output:**

```text
- **Input:** `SELECT * FROM table_name WHERE gender = 'Male';`
- **Output:** Rows where gender is Male (e.g., rows with id 1 and 3).
```

**Data:**

```json
{
  "rows": [
    {"id": 1, "gender": "Male"},
    {"id": 2, "gender": "Female"},
    {"id": 3, "gender": "Male"}
  ]
}
```

**Flow:**

1. Low-Cardinality Column
2. Bitmap Representation
3. Memory Efficiency
4. Compact Storage

