---
type: "practice"
difficulty: "Intermediate"
---

# Your warehouse bill jumped because your training jobs full-scan a huge table daily. What are the first three levers you'd pull to cut that cost?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Small Files and Compaction](<../../../../04_Data_Architecture_and_Strategy/Architecture_Patterns/Data_Architecture/Tiered_Lake_Storage/Small_Files_and_Compaction.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address the increase in storage costs due to full-table scans during ML model training, we need to focus on optimizing our data architecture and access patterns. The key areas of improvement include reducing redundant data, partitioning tables effectively, and leveraging an appropriate storage strategy.

#### Triaging the Question

The question is asking for cost-reduction strategies specifically related to training jobs that perform full table scans daily. The interviewer likely wants you to demonstrate your understanding of efficient data management practices and your ability to implement them at scale.

#### Assumptions
and Key Ideas
Current Data Architecture: We are using a traditional database system (likely a relational or columnar one) for storing training data.
Full Table Scan Issue: This suggests that the current storage strategy is not optimized, leading to unnecessary overhead during each scan.
Cost Factors: Storage costs and computational resources consumed by full table scans.

#### First-Principles Solution
Lever 1: Data Partitioning

Partitioning tables based on relevant criteria can significantly reduce the amount of data scanned in a single operation. For example, if your model is time-series dependent, partition by date or timestamp ranges could be effective. This reduces the volume of data read and ensures that only necessary partitions are processed.

Lever 2: Data Redundancy Removal

Eliminating redundant copies of data can save significant storage costs. Identify and remove duplicate entries or unnecessary snapshots. For instance, if your dataset includes historical data that is no longer needed for training, archive or delete it to free up space.

Lever 3: Storage Optimization

Switching from a traditional full-table storage to more cost-effective solutions like湖仓一体 (Lakehouse) could help. A Lakehouse combines the flexibility of data lakes with the structured querying capabilities of data warehouses. This can reduce costs by allowing you to store raw and processed data in one place, optimizing queries through schema on read.

#### Closing Summary

By implementing these strategies—partitioning tables, removing redundant data, and utilizing a cost-effective storage solution—you can significantly reduce the computational overhead associated with full table scans during training jobs. This will not only cut costs but also improve query performance and overall system efficiency.

#### Common Mistakes/Follow-ups
Overlooking Data Redundancy: Failing to identify and remove redundant copies of data.
Ignoring Storage Optimization: Not considering cost-effective storage solutions like Lakehouse or other cloud-native alternatives.
Insufficient Partitioning Strategy: Choosing the wrong partition keys, leading to inefficient query performance.

To further optimize, we might consider additional strategies such as query optimization techniques, caching frequently accessed data, and using denormalized views for reporting tasks that do not require full table scans.

**Comparison:**

| Partition Key | Interval      |
|---------------|--------------|
| timestamp     | 1 day        |

**Code:**

```python
# Example of a simple Python script for partitioning a dataset
def partition_data(df, column='timestamp', interval='1D'):
    partitions = df.groupby(pd.Grouper(key=column, freq=interval))
    return partitions

partitions = partition_data(training_df)
```

**Example input / output:**

| Input | Output |
|-------|--------|
| `training_df` (full dataset) | Partitions by date with a 30-day interval |

**Data:**

```json
{
  "table_partition": {
    "partition_column": "date",
    "partition_interval": "30D"
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Start
6. Is data redundant?
7. Remove redundant copies
8. Partition tables by relevantcriteria
9. Are there cost-effectivestorage options available?
10. Leverage Lakehouse or similarsolution
11. Optimize queries and cachingstrategies

