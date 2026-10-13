---
type: "practice"
difficulty: "Deep"
---

# Explain the difference between SCD Type 1, Type 2, and Type 3.

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 vs Mini-dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, Slowly Changing Dimensions (SCDs) are used to handle changes in dimension attributes over time. The three types of SCDs—Type 1, Type 2, and Type 3—are ways to manage these changes. Let's dive into each type from first principles.

TL;DR:
SCD Type 1: Overwrites historical data with the latest values.
SCD Type 2: Adds a new record for each change while maintaining history.
SCD Type 3: Uses additional columns to track changes over time, providing a complete history.

#### Triaging the Question

The question is asking you to explain how SCDs handle dimension changes. The interviewer will probe your understanding of data warehousing concepts and how different approaches impact data integrity and query performance. Key assumptions include familiarity with dimensional modeling and understanding that we need to track historical data accurately.

#### First-Principles Solution
1. SCD Type 1: Overwrite Historical Data
Core Idea: Replace the old record with the new one, losing all previous versions.
Use Case: Suitable when the current state is more important than history, and space constraints are a concern.

#### Example
A customer table where you only store the latest address.
SQL
-- Example of Type 1 SCD update query
UPDATE customers SET address = 'New Address' WHERE customer_id = 1;

#### Pros
Simple to implement and maintain.

#### Cons
Loses historical data, which can be crucial for detailed analysis.
2. SCD Type 2: Add a New Record with Versioning
Core Idea: Keep all versions of the record by adding a new row each time a change occurs.
Use Case: When maintaining history is essential and you have enough storage capacity.

#### Example
A customer table storing multiple addresses over time.
SQL
-- Example of Type 2 SCD insert query
INSERT INTO customers (customer_id, address, effective_date, end_date)
VALUES (1, 'New Address', '2023-10-01', NULL);

-- Marking the old record as ended
UPDATE customers SET end_date = '2023-10-01' WHERE customer_id = 1 AND address = 'Old Address';

#### Pros
Retains full history, allowing for accurate analysis over time.

#### Cons
Can lead to data bloat and increased complexity in queries.
3. SCD Type 3: Use Additional Columns for Tracking Changes
Core Idea: Track changes using additional columns within the same table row.
Use Case: When history is needed but space constraints are a concern, as this avoids multiple rows.

#### Example
A customer table with an address_version column to track changes.
SQL
-- Example of Type 3 SCD update query
UPDATE customers SET address = 'New Address', address_version = address_version + 1 WHERE customer_id = 1;

#### Pros
Combines the simplicity of Type 1 with historical data retention, reducing storage overhead.

#### Cons
Requires careful tracking and can complicate queries.

#### Closing Summary

Understanding SCD types is crucial for effective data warehousing. Each type has its strengths and weaknesses, making them suitable in different scenarios depending on business requirements and data management priorities.

#### Common Mistakes/Follow-Ups
Mistake: Assuming all SCDs are the same, leading to incorrect implementation.

#### Follow-Up
Discussing the trade-offs between space efficiency and historical accuracy, or asking about performance implications of each type.

**Comparison:**

| customer_id | address         | effective_date | end_date       |
|-------------|-----------------|----------------|---------------|
| 1           | Old Address     | 2023-09-01     | NULL           |
| 1           | New Address     | 2023-10-01     | NULL           |

**Code:**

```sql
-- Example of Type 1 SCD update query
UPDATE customers SET address = 'New Address' WHERE customer_id = 1;

-- Example of Type 2 SCD insert query
INSERT INTO customers (customer_id, address, effective_date, end_date)
VALUES (1, 'New Address', '2023-10-01', NULL);

-- Marking the old record as ended
UPDATE customers SET end_date = '2023-10-01' WHERE customer_id = 1 AND address = 'Old Address';

-- Example of Type 3 SCD update query
UPDATE customers SET address = 'New Address', address_version = address_version + 1 WHERE customer_id = 1;
```

**Example input / output:**

```text
-- Input:
UPDATE customers SET address = 'New Address' WHERE customer_id = 1;

-- Output:
Customers table updated with new address.

-- Input:
INSERT INTO customers (customer_id, address, effective_date, end_date)
VALUES (1, 'New Address', '2023-10-01', NULL);

-- Output:
Customers table inserted new record for customer_id 1.

-- Input:
UPDATE customers SET end_date = '2023-10-01' WHERE customer_id = 1 AND address = 'Old Address';

-- Output:
Old address record marked as ended.
```

**Data:**

```json
{
  "customer_id": 1,
  "address": [
    {"value": "Old Address", "effective_date": "2023-09-01", "end_date": null},
    {"value": "New Address", "effective_date": "2023-10-01", "end_date": null}
  ]
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Pros: Simple, Cons: Loss ofhistory
6. Pros: Retains full history,Cons: Data bloat
7. Pros: Combines simplicitywith history, Cons:Complicated queries
8. Pros: Full history withoutoverhead, Cons: Complexityin tracking changes
9. Type 1 SCD
10. Does space matter?
11. Overwrite old data
12. Retain history with new rows
13. Type 2 SCD
14. Space constraints?
15. Use additional columns fortracking changes
16. Add a new row per change
17. I
18. J
19. K
20. L

