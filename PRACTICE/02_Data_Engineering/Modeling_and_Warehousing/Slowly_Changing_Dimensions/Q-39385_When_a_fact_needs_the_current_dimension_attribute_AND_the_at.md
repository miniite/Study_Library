---
type: "practice"
difficulty: "Deep"
---

# When a fact needs the current dimension attribute AND the attribute at event time, how do you serve both without joining twice awkwardly?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 in Practice](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Slowly_Changing_Dimensions/SCD_Type_2_in_Practice.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR:

To handle slowly changing dimensions (SCD) where a fact requires both the current and historical values of an attribute at the time of the event, we can use a combination of SQL window functions and Common Table Expressions (CTEs). This approach allows us to avoid awkward double joins while maintaining performance and accuracy. The key idea is to leverage window functions like ROW_NUMBER() and RANK() along with CTEs to maintain the history and current state efficiently.

#### Triaging the Question

The question asks about serving both the current dimension attribute and its historical value for an event without performing awkward double joins. This scenario arises in SCD Type 2, where we track changes over time by creating a new record for each change and keeping the old records to maintain history. The challenge is to fetch these values efficiently.

#### Assumptions
We are working with a data warehouse or analytics platform.
The fact table contains events that need attributes from the dimension table.
Dimension changes are tracked using SCD Type 2.
We want to avoid performance issues associated with double joins and ensure accurate results.

#### First-Principles Solution

Identify Current and Historical Values:

For each event, identify the most recent record in the dimension table that represents the current state.
Identify the historical records corresponding to the time of the event.

Use Window Functions:

Use window functions like ROW_NUMBER() or RANK() to assign a unique identifier to each history entry based on its date/time stamp.
This helps in filtering out only relevant history entries without performing complex joins.

Common Table Expressions (CTEs):

Define CTEs for current and historical dimensions separately.
Join the fact table with these CTEs using appropriate conditions to fetch both sets of values efficiently.

Implementing the Solution:

SQL
-- Example SQL query

WITH CurrentDim AS (
  SELECT
    dim_id,
    attr1,
    attr2,
    ROW_NUMBER() OVER (PARTITION BY dim_id ORDER BY effective_date DESC) as rn
  FROM dimensions
  WHERE current_flag = 'Y'
),

HistoricalDim AS (
  SELECT
    dim_id,
    attr1,
    attr2,
    ROW_NUMBER() OVER (PARTITION BY dim_id ORDER BY effective_date DESC) as rn
  FROM dimensions
  WHERE current_flag = 'N' AND effective_date <= event_time
)

SELECT
  fact_table.event_id,
  currentDim.attr1 AS current_attr1,
  historicalDim.attr2 AS event_attr2
FROM fact_table
JOIN CurrentDim ON fact_table.dim_id = currentDim.dim_id AND rn = 1
LEFT JOIN HistoricalDim ON fact_table.dim_id = historicalDim.dim_id AND rn = 1;

#### Closing Summary
and Common Mistakes:
Common Mistake: Attempting to join the dimension table twice without using window functions or CTEs, which can lead to complex SQL queries and potential performance issues.
Best Practice: Use window functions for ranking and filtering history records directly within the query. This ensures that you only process necessary rows, reducing computational overhead.
Further Discussion:

#### Performance Considerations
Ensure that indexes are in place on effective_date and current_flag to speed up these queries.
Data Consistency:
Verify that historical records are correctly identified based on the event time for accurate results.
Scalability:
For very large datasets, consider optimizing the query by partitioning the dimension table or using materialized views.

**Comparison:**

| dim_id | attr1 | attr2 | effective_date | current_flag |
|--------|-------|-------|----------------|--------------|
| 1      | A     | B     | 2023-05-01     | Y            |
| 1      | C     | D     | 2023-06-01     | N            |
| event_id | dim_id | event_time |
|----------|--------|------------|
| 1        | 1      | 2023-05-15 |
| 2        | 1      | 2023-06-15 |

**Code:**

```sql
WITH CurrentDim AS (
  SELECT
    dim_id,
    attr1,
    attr2,
    ROW_NUMBER() OVER (PARTITION BY dim_id ORDER BY effective_date DESC) as rn
  FROM dimensions
  WHERE current_flag = 'Y'
),
HistoricalDim AS (
  SELECT
    dim_id,
    attr1,
    attr2,
    ROW_NUMBER() OVER (PARTITION BY dim_id ORDER BY effective_date DESC) as rn
  FROM dimensions
  WHERE current_flag = 'N' AND effective_date <= event_time
)
SELECT
  fact_table.event_id,
  currentDim.attr1 AS current_attr1,
  historicalDim.attr2 AS event_attr2
FROM fact_table
JOIN CurrentDim ON fact_table.dim_id = currentDim.dim_id AND rn = 1
LEFT JOIN HistoricalDim ON fact_table.dim_id = historicalDim.dim_id AND rn = 1;
```

**Example input / output:**

```text
Input:
CurrentDim CTE with:
- dim_id: 1, attr1: 'A', current_flag: 'Y'
- dim_id: 1, attr1: 'C', current_flag: 'N'

HistoricalDim CTE with:
- dim_id: 1, attr2: 'B', effective_date: '2023-05-01'
- dim_id: 1, attr2: 'D', effective_date: '2023-06-01'

Fact_table rows:
- event_id: 1, dim_id: 1, event_time: '2023-05-15'
- event_id: 2, dim_id: 1, event_time: '2023-06-15'

Output:
event_id: 1
current_attr1: A
event_attr2: B

event_id: 2
current_attr1: C
event_attr2: D
```

**Data:**

```json
{
  "dimensions": [
    {
      "dim_id": 1,
      "attr1": "A",
      "attr2": "B",
      "effective_date": "2023-05-01",
      "current_flag": "Y"
    },
    {
      "dim_id": 1,
      "attr1": "C",
      "attr2": "D",
      "effective_date": "2023-06-01",
      "current_flag": "N"
    }
  ],
  "fact_table": [
    {
      "event_id": 1,
      "dim_id": 1,
      "event_time": "2023-05-15"
    },
    {
      "event_id": 2,
      "dim_id": 1,
      "event_time": "2023-06-15"
    }
  ]
}
```

**Flow:**

1. Fact Table
2. Dimension Current
3. Rank by Effective Date
4. Filter Current Flag = 'Y'
5. Dimension History
6. Rank by Effective Date
7. Filter Current Flag = 'N' ANDeffective_date <= event_time
8. Join Fact Table with Currentand History

