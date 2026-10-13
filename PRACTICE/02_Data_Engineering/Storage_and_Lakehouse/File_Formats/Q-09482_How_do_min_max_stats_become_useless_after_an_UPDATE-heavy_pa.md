---
type: "practice"
difficulty: "Deep"
---

# How do min/max stats become useless after an UPDATE-heavy pattern of rewrites, and how would you restore effective pruning?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Parquet Internals](<../../../../02_Data_Engineering/Storage_and_Lakehouse/File_Formats/Parquet_Internals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Let's dive into how min/max statistics can become ineffective in columnar storage formats like Parquet when dealing with frequent updates and how to effectively manage these scenarios.

#### Triaging the Question

The core of this question revolves around understanding the limitations of min/max statistics in columnar file formats such as Parquet, ORC, or Avro. Specifically, it explores why these statistics can become useless during high-frequency update operations and how we can restore their effectiveness for efficient query execution.

Key Assumptions
We are working with a dataset that is frequently updated.
The focus is on maintaining the effectiveness of pruning techniques like predicate pushdown, which relies heavily on min/max statistics to filter out irrelevant data early in the query execution process.
The goal is to ensure that queries run efficiently even after multiple updates.

#### First Principles Solution
Understanding Min/Max Statistics

Min/max statistics are computed for each column and stored alongside the data. They represent the minimum and maximum values found in a specific column, which can be used by query planners to determine if a given predicate can potentially prune rows from a scan operation. For example, if you have an age column with min/max stats of 18-65, a query filtering for age > 70 can use this information to avoid scanning all data.

The Problem with Frequent Updates

When the dataset undergoes frequent updates, the min and max values stored in the statistics can quickly become stale. Consider an employee_salary column where salaries are constantly being adjusted due to promotions or bonuses. Over time, the min/max stats might no longer accurately reflect the current distribution of salary values.

#### Cons
equences on Pruning

As a result, query optimizers that rely on these statistics for pruning will be less effective. For instance, if a query filters employee_salary > 10000, and the min/max stats are outdated, the query planner might still attempt to scan the entire table, leading to suboptimal performance.

Restoring Effective Pruning

To address this issue, we need to ensure that our statistics remain up-to-date or implement alternative strategies for efficient pruning.

Updating Statistics

One approach is to regularly update the min/max statistics as part of the data refresh process. This can be done in a background job where the latest values are recomputed and stored back into the metadata. However, this might not be feasible with real-time or high-frequency updates due to potential overhead and complexity.

Hybrid Approaches

Another strategy is to use hybrid methods that combine static min/max statistics with dynamic sampling. For example:

Static Min/Max Stats: Store initial computed min/max values as a fallback.
Dynamic Sampling: Periodically take small samples from the data to update or refine these static stats.
Materialized Views and Summaries

Using materialized views can also help. These are precomputed summaries of the data that include up-to-date statistics for columns frequently used in filters. They act as a buffer between raw data and query execution, ensuring that queries operate on fresh statistics without needing to recompute them each time.

#### Closing Summary
and Common Mistakes

In summary, min/max stats can become ineffective due to frequent updates, leading to suboptimal pruning during query execution. To maintain effective pruning, regular updates or hybrid strategies like dynamic sampling should be employed. Additionally, using materialized views can provide a buffer against staleness in statistics.

#### Common mistakes
include ignoring the impact of frequent updates on statistics and relying too heavily on outdated min/max values, which can lead to poor performance in large-scale data warehousing environments.

Common Follow-Ups
How do different database systems handle these issues?
What are the trade-offs between maintaining up-to-date stats vs. computing them on-demand?
Are there any specific tools or libraries you would recommend for managing statistics in such scenarios?

**Comparison:**

| Column | Min | Max |
|--------|-----|-----|
| salary | 2000 | 150000 |
| age    | 22   | 68   |

**Code:**

```python
# Example of a simple script to refresh min/max stats
import pandas as pd

def update_min_max_stats(data):
    # Calculate min and max values for each column
    col_stats = data.describe().T[['min', 'max']]
    return col_stats.to_dict()

data = pd.read_csv('employee_data.csv')
stats = update_min_max_stats(data)
print(stats)
```

**Example input / output:**

```text
Input Query:
SELECT * FROM employees WHERE salary > 10000;

Current Statistics (Stale):
- Min: 2000, Max: 50000

Optimal Query Execution with Fresh Stats:
- Min: 2000, Max: 150000
```

**Data:**

```json
{
  "employee_salary": {
    "min": 2000,
    "max": 150000
  },
  "age": {
    "min": 22,
    "max": 68
  }
}
```

**Flow:**

1. Initial Data Load
2. Compute Min/Max Stats
3. Store Stats Metadata
4. Data Update Triggered
5. Background Job to RefreshStats
6. Stats Updated, PruningImproved
7. Query Execution with FreshStats

