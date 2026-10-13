---
type: "practice"
difficulty: "Deep"
---

# How do you decide the surrogate-key type (INT vs BIGINT) for a fact expected to reach tens of billions of rows, and what is the join-performance implication?

**Difficulty:** Deep · **Levels:** Architect/Principal, SDE-1/2, SDE-2/3 · [↑ Aggregate Fact Tables](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Aggregate_Fact_Tables.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When deciding between INT and BIGINT as the surrogate key type for a fact table in a data warehouse that expects to grow to tens of billions of rows, it's crucial to balance storage efficiency with performance implications, particularly around join operations. The primary considerations include the scale of the data, expected growth rate, potential for joins, and storage costs.

#### Triage
Understanding the Question

The question revolves around selecting an appropriate surrogate key type (either INT or BIGINT) based on the expected scale of a fact table in a data warehouse. The key aspect is to ensure that the chosen key can accommodate the projected number of rows without leading to performance issues, especially during join operations.

#### First-Principles Solution

Determine the Key Type Based on Scale:

INT (32-bit integer) has a range from -2,147,483,648 to 2,147,483,647.
BIGINT (64-bit integer) has a much larger range, from -9,223,372,036,854,775,808 to 9,223,372,036,854,775,807.

Given the expected scale of tens of billions of rows, an INT key is insufficient as it will overflow. Therefore, a BIGINT should be used.

#### Performance Considerations

Storage Efficiency: INT uses 4 bytes while BIGINT uses 8 bytes.
For a table with tens of billions of rows, using BIGINT will result in an additional 16 GB (assuming 8 bytes per row) or more, which could impact storage costs and overall performance if not justified by the use case.
Join Operations: Joins involving surrogate keys can significantly affect query performance. Larger key sizes generally have a minor impact on join operations as modern databases efficiently handle such data types.

Cost-Benefit Analysis:

If the additional 4 bytes per row is negligible compared to the overall storage cost and the business case justifies higher storage costs, then BIGINT should be chosen.
Conversely, if strict storage optimization is critical and the table’s scale can still fit within an INT, INT might be a better choice.

#### Closing Summary

In summary, for a fact table expected to reach tens of billions of rows, using BIGINT as the surrogate key type ensures that the data can accommodate the projected scale without running into overflow issues. While this increases storage requirements slightly, it is essential to balance with potential performance impacts and business constraints.

#### Common Mistakes
and Follow-Ups
Mistake: Choosing INT for a large-scale fact table.

#### Follow-Up
Question: How would you optimize join operations in such scenarios?
Mistake: Overlooking the storage cost implications of using BIGINT.

#### Follow-Up
Question: What other factors should be considered when optimizing data storage in a data warehouse?

**Code:**

```sql
-- Example SQL code for creating a fact table with BIGINT as surrogate key
CREATE TABLE sales_fact (
    sales_id BIGINT PRIMARY KEY,
    product_id INT,
    customer_id INT,
    quantity INT,
    sale_amount DECIMAL(10, 2),
    sale_date TIMESTAMP
);
```

