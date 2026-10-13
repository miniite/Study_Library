---
type: "practice"
difficulty: "Deep"
---

# Compare a BRIN index and a B-tree on an append-only time-ordered column. On what physical layout does BRIN win, and how does clustering affect its effectiveness?

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Leftmost Prefix Rule](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Leftmost_Prefix_Rule.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To compare a BRIN (Block Range Index) index with a B-tree index in the context of an append-only time-ordered column, we need to understand their underlying mechanisms, strengths, and weaknesses. A BRIN index is designed for large datasets where not all rows are scanned during query execution, whereas a B-tree index is more suited for frequent updates and small data sets.

BRIN wins when the dataset size is significantly larger than the block size (e.g., hundreds of GB or TB), as it can provide substantial performance benefits by summarizing and sampling data blocks. Clustering affects its effectiveness because well-clustered data leads to better summary accuracy, which in turn improves query performance. Conversely, poorly clustered data reduces these advantages.

#### Triaging the Question

The core question here is to understand when BRIN outperforms B-tree indices specifically for append-only time-ordered columns and how clustering impacts this. We need to consider factors like dataset size, block sizes, and the degree of data clustering.

#### First Principles Solution

BRIN Index Mechanics:

BRIN indexes are designed for large datasets where full table scans are infrequent.
They work by creating an index on a subset of blocks rather than all rows, summarizing each block's content to provide fast approximations.

B-tree Index Mechanics:

B-trees are more traditional and work well with frequent updates and smaller data sets.
They maintain sorted order and allow for efficient search operations within the index itself.

Append-Only Time-Ordered Column Considerations:

Append-only means new rows are added to the end of the table, not in the middle.
Time-ordering implies that queries often look at recent data first, which aligns with BRIN's strengths since it can quickly skip over old blocks.

Effectiveness Under Clustering:

Well-Clustered Data: If the time-ordered column has a natural clustering pattern (e.g., recent rows are grouped together), BRIN can leverage this to sample more relevant blocks, improving query performance.
Poorly Clustered Data: Poor clustering means that sampling will be less accurate, diminishing BRIN’s benefits.

#### Closing Summary
and Common Mistakes
BRIN wins when the dataset size is enormous (hundreds of GB or TB) and queries are infrequent full table scans. It outperforms B-tree indices in these scenarios due to its ability to summarize data blocks.
Clustering significantly affects BRIN's effectiveness: well-clustered data improves query performance, while poorly clustered data diminishes it.

#### Common Mistakes
Overlooking Dataset Size: Assuming B-tree is always better for large datasets.
Neglecting Query Patterns: Failing to recognize that frequent full table scans are common in append-only scenarios.
Misunderstanding Clustering Impact: Not considering how data clustering impacts BRIN’s summary accuracy.

**Comparison:**

```
Index Type	Fill Factor	Depth	Block Size
B-tree	90	4	N/A
BRIN	90	3	8192
```

**Code:**

```sql
-- Example B-tree index creation for a time-ordered column
CREATE INDEX idx_time_ordered ON my_table(time_column);

-- Example BRIN index creation (assuming block size of 8MB)
CREATE INDEX brin_idx_time_ordered ON my_table(time_column) INCLUDE (other_columns) WITH (FILLFACTOR = 90, DEPTH = 3, BLOCK_SIZE = 8192);
```

**Example input / output:**

```text
Input:
- Dataset size: 1 TB with time-ordered append-only columns.
- Query pattern: Frequent full table scans for recent data.

Output:
- B-tree index performance: Poor, due to frequent updates and large dataset.
- BRIN index performance: Good, as it can quickly skip over old blocks and provide approximate results.
```

**Data:**

```json
{
  "table": {
    "name": "my_large_table",
    "columns": [
      { "name": "time_column", "type": "timestamp" },
      { "name": "value_column", "type": "int" }
    ],
    "row_count": 10_000_000,
    "block_size": 8388608
  },
  "index_types": [
    { "name": "B-tree", "fillfactor": 90, "depth": 4 },
    { "name": "BRIN", "fillfactor": 90, "depth": 3, "block_size": 8192 }
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Append-Only Time-OrderedColumn
6. BRIN Index
7. B-tree Index
8. Is the dataset size enormous?
9. BRIN wins due to blocksummarization
10. B-tree performs better withfrequent updates
11. Are queries frequently fulltable scans?
12. BRIN outperforms B-tree byskipping old blocks
13. B-tree maintains sorted orderfor efficient searches

