---
type: "practice"
difficulty: "Expert"
---

# Your ORC-based Hive tables are still slow on selective filters. How do you use predicate pushdown, bloom filters, and stripe/row-group sizing to speed them up?

**Difficulty:** Expert · **Levels:** SDE-1/2, SDE-2/3 · [↑ Hive Partitioning and Bucketing](<../../../../TOOLS/Hadoop/Hadoop_Ecosystem/Hive_Partitioning_and_Bucketing.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the issue of slow query performance in Hive ORC-based tables with selective filters, we need to optimize the storage format and query execution process through predicate pushdown, bloom filters, and stripe/row-group sizing. Let’s break down each component step by step.

Firstly, predicate pushdown is a technique where filtering conditions are pushed as far down into the data sources as possible. Bloom filters help in reducing the amount of data read from storage by quickly identifying which blocks do not contain relevant data. Finally, stripe and row-group sizing can significantly enhance query performance by minimizing the amount of data that needs to be scanned.

**Comparison:**

| Filter Type | Explanation | Impact |
|-------------|-------------|--------|
| Predicate Pushdown | Moves filtering conditions closer to the storage layer. | Reduces the amount of data read from disk. |
| Bloom Filters | Provides a quick way to exclude irrelevant blocks. | Decreases I/O operations and improves query performance. |
| Stripe/Row-Group Sizing | Determines how data is partitioned for efficient access. | Minimizes the number of rows read, enhancing performance. |

**Code:**

```java
// Example of predicate pushdown using Hive SQL
SELECT column1, column2 FROM orc_table WHERE column3 > 10 AND column4 < 50;
```

**Example input / output:**

```text
Input: SELECT column1, column2 FROM orc_table WHERE column3 > 10 AND column4 < 50;

Output (Dry Run):
- Predicate Pushdown filters the row groups based on `column3 > 10` and `column4 < 50`.
- Bloom filter checks each ORC block to determine if it contains any relevant data.
- Only blocks containing matching rows are read, significantly reducing I/O operations.
```

**Data:**

```json
{
  "orcTable": {
    "columns": ["column1", "column2", "column3", "column4"],
    "filters": [
      { "type": "BloomFilter", "threshold": 0.01 },
      { "type": "PredicatePushdown", "conditions": ["> 10", "< 50"] }
    ]
  }
}
```

**Flow:**

1. Yes
2. No
3. Predicate Pushdown
4. Bloom Filters
5. Are filters met?
6. Read relevant ORC blocks
7. Skip irrelevant blocks
8. Return results

