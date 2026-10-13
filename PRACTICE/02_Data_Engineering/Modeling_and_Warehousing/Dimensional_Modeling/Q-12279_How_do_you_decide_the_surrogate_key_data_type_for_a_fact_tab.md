---
type: "practice"
difficulty: "Deep"
---

# How do you decide the surrogate key data type for a fact table expected to reach tens of billions of rows, and what are the join-performance implications?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Aggregate Fact Tables](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Aggregate_Fact_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When designing a database schema for a fact table that is expected to scale to tens of billions of rows, choosing the right surrogate key data type is crucial not only for storage efficiency but also for performance during joins. The primary considerations are the range and uniqueness of potential values, storage requirements, and query performance.

#### Triaging the Question

The question revolves around selecting an appropriate data type for a surrogate (primary) key in a fact table, which will be used extensively in join operations. The focus is on performance implications, specifically how different data types affect join performance. Key assumptions include:

The database system is relational.
The fact table contains billions of rows.
Join operations with dimension tables are frequent.

#### First-Principles Solution

The choice of the surrogate key data type depends on several factors:

Range and Uniqueness: A wide range of values or a low cardinality may necessitate larger data types to avoid overflow errors.
Storage Efficiency: Smaller data types require less storage, which can be critical in large datasets.
Performance Impact: Larger data types can reduce join performance due to increased memory and CPU usage during operations.

For a fact table expected to reach tens of billions of rows:

Integers (INT or BIGINT): These are commonly used for primary keys because they offer a good balance between range, storage efficiency, and performance.
BLOBs: These might be unnecessary and could complicate join operations due to their variable size.
Reasoning Step-by-Step

Identify the Range of Values:

For billions of rows, we need at least an INT (which supports a range from -2^31 to 2^31 - 1) or preferably a BIGINT if higher precision is needed.

Storage Efficiency:

An INT uses 4 bytes.
A BIGINT uses 8 bytes.
For large tables, using an INT can save significant storage space.

#### Performance Considerations

Smaller data types like INT require less memory and CPU during joins.
Larger data types like BIGINT may increase the size of join operations, leading to slower performance if not necessary.

#### Closing Summary

In conclusion, for a fact table expected to reach tens of billions of rows, using an INT or even BIGINT is generally preferred. The storage savings and performance benefits of these smaller data types far outweigh any potential risks associated with range overflow. It's essential to test the chosen data type in your specific environment to ensure optimal performance.

#### Common Mistakes/Follow-ups

Mistake: Choosing a BLOB or larger data types for primary keys.

Explanation: This can lead to unnecessary storage costs and decreased join performance.

#### Follow-up
Question: What if the fact table has a high cardinality (many unique values)?

Answer: In such cases, consider using BIGINT or even custom UUIDs (though they are less efficient for joins). The trade-off is higher storage usage but better uniqueness.

**Comparison:**

| Column    | Data Type   |
|-----------|------------|
| sale_id   | BIGINT     |
| product_id| INT        |
| store_id  | INT        |
| sales_amount| DECIMAL(10,2)|
| sale_date | DATE       |

**Code:**

```sql
-- Example of creating a fact table with BIGINT as the surrogate key
CREATE TABLE sales_fact (
    sale_id BIGINT PRIMARY KEY,
    product_id INT NOT NULL,
    store_id INT NOT NULL,
    sales_amount DECIMAL(10, 2) NOT NULL,
    sale_date DATE NOT NULL
);
```

**Example input / output:**

```text
-- Example of a join operation with the fact table and dimension tables
SELECT s.product_id, p.product_name, SUM(s.sales_amount) as total_sales
FROM sales_fact s
JOIN products_dim p ON s.product_id = p.product_id
GROUP BY s.product_id, p.product_name;
```

**Data:**

```json
{
  "sale_id": "BIGINT",
  "product_id": "INT",
  "store_id": "INT",
  "sales_amount": "DECIMAL(10,2)",
  "sale_date": "DATE"
}
```

**Flow:**

1. INT/BIGINT
2. BLOB
3. Define Table Schema
4. Surrogate Key Data Type?
5. Use INT or BIGINT for PrimaryKey
6. Misaligned with needs,consider alternatives

