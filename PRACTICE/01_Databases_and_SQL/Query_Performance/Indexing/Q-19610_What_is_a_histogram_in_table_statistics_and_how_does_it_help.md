---
type: "practice"
difficulty: "Deep"
---

# What is a histogram in table statistics and how does it help with skewed data?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Min Max Statistics and Data Skipping](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Min_Max_Statistics_and_Data_Skipping.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A histogram in database statistics represents a distribution of values in a column, helping to optimize queries by providing insights into data skewness. For skewed data, histograms enable more efficient index usage and query planning.

#### Triaging the Question

The question delves into understanding the concept of a histogram within table statistics, specifically how it aids in optimizing database performance for columns with skewed data distribution. We need to explore what a histogram is, its role in capturing data characteristics, and why it’s particularly useful when dealing with highly skewed data.

#### First-Principles Solution
What is a Histogram?

A histogram is a graphical representation of the distribution of values within a column of a database table. It breaks down the column's value range into buckets (or bins) and counts how many times each bucket appears in the data set. This information helps the database optimizer make better decisions about query execution.

Importance for Skewed Data

For columns with skewed data, where certain values appear much more frequently than others, histograms are especially valuable because:

They provide an accurate count of occurrences in specific ranges.
They help the query planner estimate selectivity and cost effectively.
They enable better pruning of indexes and tables during execution.
How Does It Work?
Bucketing: The column's value range is divided into buckets based on a heuristic or specified strategy.
Counting: Each bucket’s count of occurrences is recorded.
Usage in Optimization:
When creating an index, the histogram informs the database about which ranges are most common.
During query execution planning, the optimizer uses the histogram to estimate how many rows will match a given predicate.

#### Example

#### Cons
ider a customer table with a purchase_amount column. This column is highly skewed: many customers have small purchase amounts, while only a few make very large purchases. A histogram would capture these distributions:

PLAINTEXT
Histogram for 'purchase_amount':
Bucket 1: 0 - 500, Count = 95%
Bucket 2: 501 - 1000, Count = 4%
Bucket 3: 1001 - 5000, Count = 1%
Bucket 4: >5000, Count = 0.5%

This histogram informs the optimizer that queries targeting large purchase amounts are rare and can be efficiently pruned.

#### Closing Summary

Histograms provide a detailed view of data distribution in columns with skewed values. They assist in optimizing query performance by allowing the database to make more informed decisions about which indexes to use and how many rows might match certain conditions, especially for rare or frequent values.

#### Common Mistakes
& Follow-Ups
Failing to Understand Skewness: Misinterpreting histogram data can lead to suboptimal query plans. Understanding what constitutes skew is crucial.
Overlooking Index Maintenance: Histograms require periodic updates to remain accurate. Failure to maintain histograms can result in outdated optimization decisions.
Not Considering Other Metrics: While histograms are valuable, they should be used alongside other metrics like cardinality estimations and cost functions for comprehensive query optimization.

**Comparison:**

```
Customer	Purchase_Amount
1	50
2	450
...	...
9999	3500
10000	8000

Histogram:

PLAINTEXT
Copy
Bucket 1: 0 - 100, Count = 85%
Bucket 2: 101 - 1000, Count = 9.5%
Bucket 3: 1001 - 3000, Count = 4.5%
Bucket 4: >3000, Count = 1%
```

**Code:**

```sql
-- Example SQL to create a histogram (simplified)
CREATE STATISTICS histogram ON purchase_amount;
```

