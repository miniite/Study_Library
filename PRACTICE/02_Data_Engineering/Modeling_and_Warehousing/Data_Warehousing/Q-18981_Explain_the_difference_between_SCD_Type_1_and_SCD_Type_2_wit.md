---
type: "practice"
difficulty: "Intermediate"
---

# Explain the difference between SCD Type 1 and SCD Type 2 with an example.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ SCD Type 1 vs Type 2](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR: In data warehousing, SCD (Slowly Changing Dimensions) is a method used to handle changes in dimension attributes over time. SCD Type 1 simply updates the attribute value with the new one while preserving historical records, whereas SCD Type 2 keeps all history by creating new rows for each change.

#### Triage
The question asks us to explain the difference between two types of Slowly Changing Dimensions (SCD) in data warehousing: Type 1 and Type 2. It requires understanding the basic concepts and practical implications of these methods. We will assume that we are discussing a dimension table with attributes such as customer details, where changes need to be tracked over time.

Step-by-Step Solution: Let's consider a scenario where a customer’s address is updated in a dimension table used for our data warehouse.

SCD Type 1:

When an attribute (like address) of a dimension changes, the new value replaces the old one.
Historical records are not retained; only the most recent version is stored.

SCD Type 2:

A new row is created in the dimension table for each change in the attribute.
The original record remains intact, and a new record is inserted with an updated value.
This method preserves historical data but increases the size of the dimension table.

#### Example
(Type 1):

Suppose we have a customer table Customers:

CustomerID	Name	Address
101	John Doe	123 Main St

If John’s address is updated to “456 Elm St”, using SCD Type 1, the record would be modified as follows:

CustomerID	Name	Address
101	John Doe	456 Elm St

#### Example
(Type 2):

Using SCD Type 2, a new row is created for the updated address:

CustomerID	Name	Address	EffectiveDate	ExpirationDate
101	John Doe	123 Main St	2023-01-01	9999-12-31
101	John Doe	456 Elm St	2023-07-15	NULL

#### Closing Summary
SCD Type 1 is simpler and saves space by overwriting old values, but it loses historical context. SCD Type 2 preserves history at the cost of increased storage requirements.

#### Common Mistakes/Follow-Ups

Forgetting to define effective and expiration dates for SCD Type 2.
Assuming SCD Type 1 is always better because it’s simpler, which may not be true depending on business needs.
Not considering performance impacts when deciding between the two types, especially in large datasets.

For a fresher level engineer: Focus on understanding the basic concepts and simple examples. For a mid-level engineer: Discuss trade-offs and best practices for choosing SCD Type 1 or 2 based on use cases. For an architect/principal engineer: Consider scalability, storage optimization techniques, and how these methods fit into a larger data strategy.

**Comparison:**

| CustomerID | Name    | Address          | EffectiveDate   | ExpirationDate |
|------------|---------|------------------|-----------------|----------------|
| 101        | John Doe| 123 Main St      | 2023-01-01      | 9999-12-31     |
| 101        | John Doe| 456 Elm St       | 2023-07-15      | NULL           |

**Code:**

```sql
-- Example of updating with SCD Type 1
UPDATE Customers
SET Address = '456 Elm St'
WHERE CustomerID = 101;

-- Example of inserting with SCD Type 2
INSERT INTO Customers (CustomerID, Name, Address, EffectiveDate, ExpirationDate)
VALUES (101, 'John Doe', '456 Elm St', '2023-07-15', NULL);
```

**Example input / output:**

```text
Input (Type 1): Update customer address in Customers table.
Output:
| CustomerID | Name    | Address          |
|------------|---------|------------------|
| 101        | John Doe| 456 Elm St       |

Input (Type 2): Insert new row with updated address.
Output:
| CustomerID | Name    | Address          | EffectiveDate   | ExpirationDate |
|------------|---------|------------------|-----------------|----------------|
| 101        | John Doe| 123 Main St      | 2023-01-01      | 9999-12-31     |
| 101        | John Doe| 456 Elm St       | 2023-07-15      | NULL           |
```

**Data:**

```json
{
  "Customers": [
    {
      "CustomerID": 101,
      "Name": "John Doe",
      "Address": "123 Main St",
      "EffectiveDate": "2023-01-01",
      "ExpirationDate": "9999-12-31"
    },
    {
      "CustomerID": 101,
      "Name": "John Doe",
      "Address": "456 Elm St",
      "EffectiveDate": "2023-07-15",
      "ExpirationDate": null
    }
  ]
}
```

**Flow:**

1. SCD Type 1
2. Update existing row
3. SCD Type 2
4. Insert new row with updatedvalue

