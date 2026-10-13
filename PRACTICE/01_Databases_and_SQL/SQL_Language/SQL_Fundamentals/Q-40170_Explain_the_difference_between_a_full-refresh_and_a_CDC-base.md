---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between a full-refresh and a CDC-based incremental strategy for a large dimension, and how you'd decide the crossover point where incremental complexity pays off.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Schema Evolution in Lakehouse Tables](<../../../../01_Databases_and_SQL/SQL_Language/SQL_Fundamentals/Schema_Evolution_in_Lakehouse_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address this question, let’s first outline the key differences between a full-refresh (FR) and a CDC-based incremental strategy in the context of large dimensions. Then, we'll discuss how to determine the optimal crossover point where an incremental approach starts to outweigh the benefits of a full refresh.

TL;DR
Full Refresh: Involves rebuilding the entire dimension table each time, which is straightforward but may be inefficient for large datasets.
CDC-Based Incremental: Tracks changes in real-time or near-real-time and updates only the changed records, making it more efficient for large dimensions with frequent updates. However, it requires additional complexity to manage change data.

#### Triaging the Question

The interviewer is likely probing your understanding of ETL/ELT processes, particularly around dimension table maintenance strategies. They may also be interested in your ability to balance simplicity and efficiency, as well as how you handle trade-offs between different approaches based on real-world constraints.

#### Assumptions
The dimension table contains historical data.
Changes are frequent but not uniformly distributed.
Performance is a key consideration.

#### First Principles Solution
Full Refresh:

#### Pros
Simple and straightforward to implement.

#### Cons
High resource usage, particularly for large datasets.
CDC-Based Incremental:

#### Pros
Efficient use of resources by only updating changed records.

#### Cons
Increased complexity in managing change data and maintaining consistency.
Determining the Crossover Point

The decision on when to transition from a full refresh to an incremental strategy hinges on several factors:

Change Frequency:
If changes are infrequent, a full refresh might be more efficient due to lower overhead.
Dataset Size:
For very large datasets, the cost of full refresh can become prohibitive even with modern infrastructure.
Resource Constraints:
Limited compute resources may make incremental strategies necessary for efficiency.
Latency Requirements:
Real-time or near-real-time updates might necessitate incremental strategies.
Step-by-Step Reasoning

Initial Considerations:

Assess the average number of changes per update window (e.g., daily).
Estimate the cost of a full refresh versus the benefits gained from incremental updates.

Cost-Benefit Analysis:

Calculate the cost of performing a full refresh vs. the cumulative cost of maintaining an incremental strategy over time.

#### Cons
ider any additional costs such as storage for change data, or potential operational overhead.

Performance Metrics:

Monitor performance metrics (e.g., update times, resource usage) under both strategies.
Use these to dynamically adjust the approach if necessary.

Failover and Recovery:

Ensure that a full refresh can be used as a fail-safe mechanism.
Plan for scenarios where an incremental strategy might fail, requiring a fallback to a full refresh.
Practical Example

#### Cons
ider a dimension table with 1 million rows experiencing about 10,000 updates per day. A full refresh would take around 30 minutes, while incremental processing could be done in under 5 minutes if properly optimized.

Crossover Point Calculation
Full Refresh: ( T_{FR} = \frac{\text{Total Rows}}{\text{Rows per Update}} \times \text{Time per Update} )
Incremental Processing: ( T_{IPC} = \sum (\text{Change Rows}) \times \text{Time per Change} )

Where:

Total Rows = 1,000,000
Rows per Update = 1,000 (assuming 1% change)
Time per Update = 30 minutes
Time per Change = 5 seconds

#### Example
Calculation
( T_{FR} = \frac{1,000,000}{1,000} \times 30 = 30,000 ) minutes (impractical)
( T_{IPC} = 10,000 \times 5 = 50,000 ) seconds or approximately 13.89 hours

In this case, the full refresh would be more efficient if changes are infrequent enough to keep the update time manageable.

#### Closing Summary
and Common Mistakes

Deciding on a crossover point involves balancing simplicity and efficiency. A common mistake is ignoring real-world performance metrics when making decisions. Always validate assumptions through monitoring and adjust strategies based on observed behavior.

#### Follow-Up
Questions
How would you handle changes in schema or structure?
What about data quality checks during incremental updates?

**Comparison:**

```
Dimension Table (Full Refresh)	Change Data (CDC-Based)
ID	Name
1	John
2	Jane
3	Mike
```

**Code:**

```sql
-- Example SQL for full refresh
DELETE FROM dimension_table;
INSERT INTO dimension_table SELECT * FROM source_table;

-- Example SQL for CDC-based incremental update
UPDATE dimension_table SET column1 = new_value WHERE id IN (SELECT id FROM change_data WHERE column1 IS DISTINCT);
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Full Refresh: `DELETE FROM dimension_table; INSERT INTO dimension_table SELECT * FROM source_table;`<br>Incremental Update: `UPDATE dimension_table SET name = 'Jill' WHERE id IN (SELECT id FROM change_data);` | Updated Dimension Table with 10,000 records and a single update |

**Data:**

```json
{
  "dimension_table": {
    "id": [1, 2, 3],
    "name": ["John", "Jane", "Mike"],
    "last_updated": ["2023-01-01", "2023-01-02", "2023-01-03"]
  },
  "change_data": {
    "id": [2],
    "name": ["Jill"],
    "last_updated": ["2023-01-04"]
  }
}
```

**Flow:**

1. Low
2. High
3. Initial State
4. Change Frequency
5. Full Refresh
6. Incremental Update
7. Update Strategy

