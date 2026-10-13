---
type: "practice"
difficulty: "Basic"
---

# What SQL join type is most common when querying a fact table against its dimensions?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database development, understanding how to effectively query data using different types of SQL joins is crucial for optimizing performance and ensuring accurate results. When querying a fact table in a star or snowflake schema, the most common join type used is an INNER JOIN. This is because we are typically interested in retrieving only the records where there is matching data between the fact and dimension tables.

#### Triage
Restating the Question and What's Being Probed

The question asks about the SQL join type commonly used when querying a fact table against its dimensions. The key elements here involve understanding:

Star/Snowflake Schema: This refers to the structure of your database where fact tables are joined with multiple dimension tables.
Fact Table vs Dimensions: Fact tables store the primary data, while dimension tables provide context about that data.

#### First Principles Solution

To fully understand why an INNER JOIN is commonly used in this scenario, let's break it down:

INNER JOIN Definition: This join returns only the records that have matching values in both the fact and dimension tables. It’s ideal for situations where we want to see a clear relationship between the data.
Star Schema: In a star schema, one or more fact tables are surrounded by multiple dimensions, all linked through foreign keys in the fact table. The key fields here are often small (e.g., IDs) and can be efficiently compared using INNER JOINs.
Why INNER JOIN is Common
Accuracy: Inner joins ensure that only records with matching data from both sides are returned. This helps maintain data integrity.
Performance: Since dimensions usually have fewer rows than fact tables, inner joins can be faster as they reduce the number of unnecessary comparisons and results.
Simplicity: Inner joins are straightforward to implement and understand, making them a preferred choice for most queries.

#### Common Mistakes
and Follow-Ups
Misunderstanding Join Types: Often, developers might confuse INNER JOIN with other types like LEFT JOIN or FULL OUTER JOIN, which can lead to incorrect results if not properly understood.
Ignoring Data Size: Sometimes, using INNER JOIN on large tables can be inefficient. Always consider the size of your data and choose a join type that best suits performance needs.
Joining Incorrect Columns: Ensure you are joining the correct columns (foreign keys) between fact and dimension tables to avoid incorrect results.

#### Closing Summary

When querying a fact table in a star or snowflake schema, the most common SQL join type used is INNER JOIN due to its accuracy, performance benefits, and simplicity. However, always consider your specific use case, data sizes, and desired outcomes when choosing the appropriate join type.

**Comparison:**

| Fact Table | Dimension 1 | Dimension 2 |
| --- | --- | --- |
| id | dim1_id | dim2_id |
| name1 | name | name2 |

**Code:**

```sql
-- Example of an INNER JOIN in a star schema context
SELECT fact_table.*, dimension1.*, dimension2.*
FROM fact_table
INNER JOIN dimension1 ON fact_table.dim1_id = dimension1.id
INNER JOIN dimension2 ON fact_table.dim2_id = dimension2.id;
```

**Example input / output:**

```text
Input:
fact_table = [{"id": 1, "dim1_id": 101, "dim2_id": 202}, {"id": 2, "dim1_id": 103, "dim2_id": 204}]
dimension1 = [{"id": 101, "name": "DimA"}, {"id": 103, "name": "DimB"}]
dimension2 = [{"id": 202, "name": "DimX"}, {"id": 204, "name": "DimY"}]

Output:
[{"id": 1, "dim1_id": 101, "dim2_id": 202, "name": "DimA", "name2": "DimX"},
 {"id": 2, "dim1_id": 103, "dim2_id": 204, "name": "DimB", "name2": "DimY"}]
```

**Data:**

```json
{
  "fact_table": [
    {"id": 1, "dim1_id": 101, "dim2_id": 202},
    {"id": 2, "dim1_id": 103, "dim2_id": 204}
  ],
  "dimension1": [
    {"id": 101, "name": "DimA"},
    {"id": 103, "name": "DimB"}
  ],
  "dimension2": [
    {"id": 202, "name": "DimX"},
    {"id": 204, "name": "DimY"}
  ]
}
```

**Flow:**

1. Fact Table
2. Dimension 1
3. Dimension 2
4. Join Condition
5. Result

