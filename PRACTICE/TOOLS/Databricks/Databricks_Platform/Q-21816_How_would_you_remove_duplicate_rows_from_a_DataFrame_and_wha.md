---
type: "practice"
difficulty: "Basic"
---

# How would you remove duplicate rows from a DataFrame and what is the difference between distinct and dropDuplicates?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Databricks Query Performance](<../../../../TOOLS/Databricks/Databricks_Platform/Databricks_Query_Performance.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address how to remove duplicate rows from a DataFrame in Databricks, we need to understand two key methods: distinct() and dropDuplicates(). Both serve the purpose of eliminating duplicates, but they have subtle differences. Let's break down each method step by step.

#### Triaging the Question

The question asks us to explain how to remove duplicate rows from a DataFrame in Databricks and compare distinct() with dropDuplicates(). The interviewer is probing your understanding of common data manipulation techniques in Databricks, as well as your ability to articulate differences between similar functionalities. We need to assume that the DataFrame contains multiple columns.

#### First-Principles Solution

Understanding distinct():

distinct() returns a new DataFrame with duplicate rows removed based on all columns.
It is equivalent to performing an SQL SELECT DISTINCT operation, which removes duplicates by considering every column in the DataFrame.

Understanding dropDuplicates():

dropDuplicates() also returns a DataFrame but allows you to specify the columns based on which duplicates should be removed.
By default, it considers all columns for deduplication.

#### Closing Summary
+ Common Mistakes/Follow-Ups

In summary, both distinct() and dropDuplicates() are used to remove duplicate rows in Databricks. However, distinct() operates on the entire DataFrame, while dropDuplicates() allows you to specify which columns to consider for duplicates. Using dropDuplicates() is more flexible as it lets you define the criteria for deduplication.

#### Common mistakes
include not considering edge cases where only a subset of columns needs unique values or assuming that distinct() and dropDuplicates() are exactly the same, which can lead to incorrect results if used incorrectly.

Common Edge Cases
Empty DataFrame: Both methods should gracefully handle an empty DataFrame.
Single Column Deduplication: When using dropDuplicates(), ensure you specify at least one column. If no columns are specified, it will behave similarly to distinct() considering all columns.
Closing Notes

Both functions are essential for data preprocessing and cleaning in Databricks. Understanding their differences can help you choose the right method based on your specific requirements.

**Comparison:**

| DataFrame | distinct()       | dropDuplicates(['A']) |
|-----------|------------------|----------------------|
| Behavior  | All columns for deduplication | Specified column 'A' only |
| Example Output | +---+---+   | +---+---+    |
|             | |  A | B  |   | |  A | B  |
|             | +---+---+   | +---+---+    |
|             | | 1 | 2  |   | | 1 | 2  |
|             | +---+---+   | +---+---+    |
|             | | 3 | 4  |   | | 3 | 4  |
|             | +---+---+   | +---+---+    |
|             | | 5 | 6  |   | | 5 | 6  |
|             | +---+---+   | +---+---+    |
|             | | 7 | 8  |   | | 7 | 8  |
|             | +---+---+   | +---+---+    |

**Example input / output:**

```text
Input DataFrame:
+---+---+
|  A |  B|
+---+---+
|  1 |  2|
|  3 |  4|
|  5 |  6|
|  7 |  8|
|  1 |  9|
+---+---+

distinct_df:
+---+---+
|  A |  B|
+---+---+
|  1 |  2|
|  3 |  4|
|  5 |  6|
|  7 |  8|
+---+---+

drop_duplicates_df (specified column 'A'):
+---+---+
|  A |  B|
+---+---+
|  1 |  2|
|  3 |  4|
|  5 |  6|
|  7 |  8|
+---+---+
```

**Flow:**

1. Yes
2. No
3. DataFrame
4. distinct
5. Specified Columns?
6. dropDuplicates(['A'])
7. distinct all columns
8. Output: Deduplicated basedon 'A'
9. Output: Deduplicated basedon all columns

