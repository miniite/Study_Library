---
type: "practice"
difficulty: "Intermediate"
---

# What is an upsert (MERGE) statement and how is it used when loading dimension tables?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ SCD Type 1 vs Type 2](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

An UPSERT or MERGE statement in data warehousing is a powerful tool for handling insertions and updates to dimension tables efficiently. It essentially combines an INSERT with an UPDATE, allowing you to add new records while also updating existing ones based on certain conditions. This approach ensures that your dimensional model remains accurate and up-to-date without the need for complex logic.

#### Triage
Understanding the Question

The question asks about the concept of an UPSERT or MERGE statement, specifically in the context of loading dimension tables. The key points to address are:

What is an UPSERT/MERGE statement?
How does it work?
Why use UPSERT/MERGE for dimension table updates?

#### First-Principles Solution
What is an UPSERT/MERGE Statement?

An UPSERT (short for "UPDATE or INSERT") or MERGE statement allows you to either insert new records into a database table, or update existing ones if they already exist. This operation is particularly useful when dealing with dimension tables in data warehousing.

How Does It Work?

When using an UPSERT/MERGE statement:

Matching Condition: You define a condition that specifies which rows should be considered for updating.
Insertion: If no matching row exists, the new record is inserted into the table.
Update: If a matching row does exist, it is updated with the new values.
Why Use UPSERT/MERGE for Dimension Table Updates?
Efficiency: Avoids unnecessary full scans of the table by leveraging index conditions to find existing records quickly.
Maintaining Data Integrity: Ensures that only valid and up-to-date data persists in your dimension tables, reducing the risk of inconsistencies.
Flexibility: Can handle both insertions and updates with a single statement, simplifying the ETL (Extract, Transform, Load) process.

#### Example Scenario

Suppose we have a dimension table for customers called DIM_CUSTOMERS, which contains columns like customer_id, name, address, etc. We want to load new customer data while updating existing records if necessary.

SQL
MERGE INTO DIM_CUSTOMERS AS TARGET
USING (SELECT * FROM STAGING_CUSTOMERS) AS SOURCE
ON TARGET.customer_id = SOURCE.customer_id
WHEN MATCHED THEN
    UPDATE SET
        TARGET.name = SOURCE.name,
        TARGET.address = SOURCE.address
WHEN NOT MATCHED THEN
    INSERT (customer_id, name, address)
    VALUES (SOURCE.customer_id, SOURCE.name, SOURCE.address);
Step-by-Step Breakdown:
MATCHED: If a customer ID exists in both the TARGET and SOURCE, update their details.
NOT MATCHED: Insert new records from the staging area into DIM_CUSTOMERS.

#### Closing Summary

In summary, an UPSERT/MERGE statement is a versatile SQL command that combines the functionality of INSERT and UPDATE. It ensures efficient data loading by handling both insertions and updates in a single operation. This approach is particularly valuable for maintaining accurate dimension tables in data warehousing.

#### Common Mistakes/Follow-Ups
Incorrect Matching Condition: Failing to correctly define the condition for matching rows can lead to unintended updates or inserts.
Performance Issues: Large datasets may require optimizing index usage and partitioning strategies to ensure efficient execution of MERGE statements.
Concurrency Considerations: Ensure that concurrent operations do not interfere with the integrity of your data.

**Code:**

```sql
-- Example UPSERT statement for dimension table
MERGE INTO DIM_CUSTOMERS AS TARGET
USING (SELECT * FROM STAGING_CUSTOMERS) AS SOURCE
ON TARGET.customer_id = SOURCE.customer_id
WHEN MATCHED THEN
    UPDATE SET
        TARGET.name = SOURCE.name,
        TARGET.address = SOURCE.address
WHEN NOT MATCHED THEN
    INSERT (customer_id, name, address)
    VALUES (SOURCE.customer_id, SOURCE.name, SOURCE.address);
```

**Example input / output:**

```text
Input: UPSERT statement for DIM_CUSTOMERS

Output:
- Existing record with customer_id 1 updated to new address.
- New record with customer_id 3 inserted into DIM_CUSTOMERS.
```

**Data:**

```sql
-- Sample data for DIM_CUSTOMERS and STAGING_CUSTOMERS tables
DIM_CUSTOMERS:
(customer_id, name, address)
(1, 'John Doe', '123 Main St')
(2, 'Jane Smith', '456 Elm St')

STAGING_CUSTOMERS:
(customer_id, name, address)
(1, 'John Doe', '123 New St')
(3, 'Alice Johnson', '789 Oak St')
```

