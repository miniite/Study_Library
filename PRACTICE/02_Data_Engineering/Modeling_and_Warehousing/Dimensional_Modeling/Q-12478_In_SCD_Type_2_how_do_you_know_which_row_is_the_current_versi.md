---
type: "practice"
difficulty: "Intermediate"
---

# In SCD Type 2, how do you know which row is the current version of a customer?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 vs Mini-dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In a Star or Snowflake schema, data is organized around fact tables with dimension tables branching out from them. SCD Type 2 (Slowly Changing Dimension Type 2) allows you to manage changes in dimension records by creating new rows instead of updating existing ones. To determine the current version of a customer in an SCD Type 2 setup, we need to understand how these new rows are marked and stored.

#### Triage

The question asks about identifying the current version of a customer using SCD Type 2. The key here is understanding the metadata fields used to track history, specifically the effective date range (start_date and end_date) which help us determine the validity period of each record.

#### Solution

Understanding SCD Type 2: In an SCD Type 2 approach, when a change occurs in a customer's dimension data (e.g., address update), you insert a new row with a different surrogate key and mark it as active. The previous version is marked as inactive by setting the end_date to the current timestamp.

Metadata Fields:

surrogate_key: A unique identifier for each customer record.
start_date: Indicates when the record became effective.
end_date: Indicates when the record was made inactive (or remains NULL if still active).

Querying the Current Version: To find the current version of a customer, you can use a SQL query that filters records where the end_date is either NULL or a future date compared to the current timestamp.

SQL
SELECT *
FROM customers
WHERE surrogate_key = 'customer_123'
  AND (end_date IS NULL OR end_date > CURRENT_TIMESTAMP);

Handling Version Control: Ensure that your table design includes these metadata fields and that they are updated correctly during transactions.

#### Example Scenario

Suppose a customer's address changes on 2023-10-01.
The first row has start_date as 2023-10-01 and is marked active with end_date NULL.
A new row is inserted with the same surrogate key, start_date set to 2023-10-02, and end_date set to 9999-12-31 (indicating it's currently active).
The previous row has its end_date updated to 2023-10-01.

#### Closing Summary

In summary, identifying the current version of a customer in SCD Type 2 involves using metadata fields such as start_date and end_date. By querying rows where these fields indicate an active period (i.e., end_date is NULL or future), you can determine which record represents the most recent state.

#### Common Mistakes/Follow-Ups
Overwriting Instead of Inserting: One common mistake is accidentally updating existing records instead of inserting new ones.
Inconsistent Date Handling: Ensure that start_date and end_date are consistently updated to avoid overlapping periods or gaps in the data.

If you need to implement this, consider testing with a sample dataset to ensure correctness.

**Comparison:**

```
table {
    headers: ["surrogate_key", "name", "address.street", "start_date", "end_date"]
    rows: ["customer_123", "John Doe", "456 Oak St", "2023-10-02", null]
}
```

**Code:**

```sql
-- Example SQL query for finding current customer version
SELECT *
FROM customers
WHERE surrogate_key = 'customer_123'
  AND (end_date IS NULL OR end_date > CURRENT_TIMESTAMP);
```

**Example input / output:**

```text
-- Input query to find current version of customer_123
SELECT *
FROM customers
WHERE surrogate_key = 'customer_123'
  AND (end_date IS NULL OR end_date > CURRENT_TIMESTAMP);

-- Output for a non-current row
| surrogate_key | name     | address.street | ... | start_date | end_date |
|---------------|----------|----------------|-----|-----------|---------|
| customer_123   | John Doe | 123 Elm St     | ... | 2023-10-01 | NULL    |

-- Output for a current row
| surrogate_key | name     | address.street | ... | start_date | end_date |
|---------------|----------|----------------|-----|-----------|---------|
| customer_123   | John Doe | 456 Oak St     | ... | 2023-10-02 | NULL    |
```

**Data:**

```json
{
  "surrogate_key": "customer_123",
  "name": "John Doe",
  "address": {
    "street": "123 Elm St",
    "city": "Anytown",
    "state": "CA"
  },
  "start_date": "2023-10-01",
  "end_date": null
}
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Customer Record Update
6. Is the update a change in keyfields?
7. Insert new row with currenttimestamp as start_date
8. Update existing record withend_date set to currenttimestamp?
9. Do nothing
10. Mark existing record asinactive by updating itsend_date
11. Insert new row with updatedfields and current timestampas start_date

