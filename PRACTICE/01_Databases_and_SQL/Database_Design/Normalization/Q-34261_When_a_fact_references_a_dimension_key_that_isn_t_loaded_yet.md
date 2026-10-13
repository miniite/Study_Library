---
type: "practice"
difficulty: "Deep"
---

# When a fact references a dimension key that isn't loaded yet do you use a late-arriving dimension placeholder or defer the row? Walk through the trade-off.

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Referential Integrity Checks](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Referential_Integrity_Checks.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When dealing with the situation where a fact table references a dimension key that hasn’t been loaded yet, you have two primary options: using a late-arriving dimension placeholder or deferring the row until the necessary data is available. Both approaches come with their own trade-offs and implications for data integrity and performance.

To clarify, let's define our scenario:

Late-Arriving Dimension Placeholder: You insert a placeholder in the fact table when you initially load it. This placeholder serves as a temporary entry, and its value will be updated later.
Deferring the Row: You defer loading the entire row until all necessary dimension data is available.

Now, let's delve into the detailed trade-offs for each approach:

Late-Arriving Dimension Placeholder

#### Pros

Immediate Data Availability: By inserting a placeholder, you ensure that your fact table remains complete and can be queried immediately.
Simplified Workflow: You don’t need to wait for all data before processing, which can speed up the initial loading phase.

#### Cons

Data Inconsistency: The presence of placeholders can lead to inconsistencies. Users might query incomplete or incorrect data until the placeholder is updated.
Potential Performance Issues: If not managed correctly, a high number of placeholders could degrade query performance due to indexes and other optimizations being less effective.
Deferring the Row

#### Pros

Data Integrity: Ensures that all necessary dimension data is available before processing. This can improve data quality and reduce inconsistencies.

#### Cons
istent Query Results: Users always get complete, consistent results from queries, which enhances trust in the data.

#### Cons

Delayed Data Loading: You must wait for all related dimensions to be fully loaded before inserting into the fact table, potentially delaying critical processes.
Resource Utilization: The delay can lead to inefficiencies if multiple facts are dependent on the same missing dimension.
Step-by-Step First Principles Reasoning

Initial Load of Facts: When loading your fact data, you might encounter keys that correspond to dimensions which have not yet been loaded.

Decision Point: For each such key, you need to decide whether to:

Insert a placeholder and continue processing,
Skip the row entirely (if it’s critical),
Or defer the entire row until all necessary dimension data is available.

Impact on Query Performance: If placeholders are used, you need to handle them in your queries or ETL processes to ensure they do not skew results.

Data Warehouse Design Considerations: You must design your tables and indexes accordingly. Placeholder handling can complicate indexing strategies.

#### Example Scenario

#### Cons
ider a retail analytics fact table where each transaction references customer demographic information stored in a dimension. If you encounter a new customer ID during the initial data load, do you:

Insert a placeholder with default values (e.g., unknown for all fields) and mark it as incomplete?
Or skip this row entirely until all necessary customer details are collected?

#### Closing Summary

Both late-arriving dimension placeholders and deferring rows have their merits. Late-arriving placeholders offer immediate data availability but come with potential data inconsistencies, while deferring rows ensures consistency at the cost of delayed processing.

#### Common Mistakes/Follow-Ups
Ignoring Placeholder Management: Not having a robust strategy for managing placeholders can lead to ongoing issues and performance degradation.
Over-Optimizing Early Loads: Failing to consider long-term data integrity in favor of immediate availability can harm the overall data quality.
Failing to Test Thoroughly: Not thoroughly testing how your system handles these scenarios, particularly under high load or with incomplete data.

**Comparison:**

| Fact Table | Dimension Table |
|------------|----------------|
| transaction_id, customer_id, purchase_date | id, name, email |

**Code:**

```sql
-- Example SQL for Late-Arriving Placeholder
INSERT INTO fact_table (transaction_id, customer_id)
VALUES ('T123', 999); -- Using a placeholder customer ID

-- Example SQL for Deferring the Row
INSERT INTO fact_table (transaction_id, customer_id)
VALUES ('T456', NULL); -- Deferring until full data is available
```

**Example input / output:**

| Input | Output |
|-------|--------|
| Insert transaction `T123` with placeholder customer ID `999`. <br> Load dimension data for `customer_id` 999. | Placeholder inserted for `customer_id` 999.<br> Update fact table to use actual values after loading dimensions.

**Data:**

```json
{
    "fact": {
        "transaction_id": "T123",
        "customer_id": 999,
        "purchase_date": "2023-10-05"
    },
    "dimension": {
        "id": 999,
        "name": "Placeholder Customer",
        "email": null
    }
}
```

**Flow:**

1. Deferring Rows
2. Late-Arriving Placeholder
3. Placeholder
4. Deferral
5. Initial Load of Facts
6. Placeholder or Deferral?
7. Insert Placeholder
8. Mark Incomplete and Querywith Care
9. Wait for All Data
10. Insert Row with Full Data
11. Handle Placeholder inQueries/ETL
12. Ensure Consistency AcrossSystem
13. Delay Fact Table Loading UntilComplete
14. Maintain Data Integrity andConsistency

