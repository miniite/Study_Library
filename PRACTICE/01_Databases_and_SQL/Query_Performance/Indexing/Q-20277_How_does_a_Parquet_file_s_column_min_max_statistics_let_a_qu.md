---
type: "practice"
difficulty: "Deep"
---

# How does a Parquet file's column min/max statistics let a query engine skip row groups, and how is that like a coarse index?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Min Max Statistics and Data Skipping](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Min_Max_Statistics_and_Data_Skipping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a Parquet file, the column min/max statistics are used by a query engine to determine which row groups can be skipped during query execution, thus optimizing the overall performance of the query. This is akin to using a coarse index in traditional databases, where an index helps in quickly narrowing down the search space.

#### Triaging the Question

The question aims to understand how column min/max statistics in Parquet files facilitate efficient querying and compare this technique with indexing in relational databases. The key assumptions are that we're dealing with large datasets stored in Parquet format and that query optimization is critical for performance.

#### First-Principles Solution

Column Min/Max Statistics:

When a Parquet file is written, each column's minimum (min) and maximum (max) values are recorded.
These statistics help the query engine to determine which row groups do not contain data that could satisfy any part of the WHERE clause in the SQL query.

Query Execution Flow:

During a query execution, the engine first examines the column min/max statistics for each row group.
If no part of the WHERE clause can be satisfied by values within a particular row group's statistics (e.g., if WHERE x > 100 and all rows in this group have x <= 50), then that entire row group is skipped.

Comparison with Indexes:

In traditional databases, an index provides the same benefit of quickly identifying which parts of a table can be ignored based on key values.
However, Parquet's columnar storage and min/max statistics offer a more dynamic and efficient way to skip entire row groups without needing to read any data.

#### Closing Summary
+ Common Mistakes

Column min/max statistics in Parquet files are like coarse indexes in traditional databases; they help the query engine to quickly eliminate irrelevant parts of large datasets, thus improving performance. However, while both serve a similar purpose, there are key differences:

Granularity: Indexes can be very fine-grained and specific (e.g., covering individual columns or column combinations), whereas min/max statistics are coarser but still effective.
Dynamic Nature: Min/max stats update automatically with data changes, whereas traditional indexes might need periodic rebuilding.

#### Common Mistakes
Over-reliance on Statistics: Assuming that having good min/max statistics guarantees efficient queries without considering other factors like query complexity and data skew.
Ignoring Other Optimization Techniques: Focusing solely on column min/max stats while neglecting other optimizations such as partitioning, indexing in traditional databases, or using more advanced query execution plans.

**Comparison:**

```
Row Group ID	Column 'a' Range	Column 'b' Range
0	Min: 1, Max: 10	Min: 20, Max: 30
1	Min: 5, Max: 15	Min: 25, Max: 35
Query Column	Value
a	6
b	27
```

**Code:**

```python
# Example code to simulate skipping row groups based on statistics
def skip_row_groups(query_columns, row_group_stats):
    skipped_groups = []

    for group_id, stats in row_group_stats.items():
        # Check if any column's min/max values satisfy the query conditions
        if all(stats[col]['min'] > val or stats[col]['max'] < val for col, val in query_columns.items()):
            skipped_groups.append(group_id)

    return skipped_groups

# Example data structure
row_group_stats = {
    0: {'a': {'min': 1, 'max': 10}, 'b': {'min': 20, 'max': 30}},
    1: {'a': {'min': 5, 'max': 15}, 'b': {'min': 25, 'max': 35}}
}

query_columns = {'a': 6, 'b': 27}
skipped_groups = skip_row_groups(query_columns, row_group_stats)
print(f"Skipped Row Groups: {skipped_groups}")
```

**Example input / output:**

```text
Skipped Row Groups: [0]
```

**Data:**

```json
{
    "row_group_stats": {
        0: {"a": {"min": 1, "max": 10}, "b": {"min": 20, "max": 30}},
        1: {"a": {"min": 5, "max": 15}, "b": {"min": 25, "max": 35}}
    },
    "query_columns": {
        "a": 6,
        "b": 27
    }
}
```

**Flow:**

1. Examine Min/Max Stats
2. Skip if No Match
3. Proceed to Read Data
4. Start
5. Query Execution
6. C
7. End
8. (Read Row Groups)
9. Skipped Groups

