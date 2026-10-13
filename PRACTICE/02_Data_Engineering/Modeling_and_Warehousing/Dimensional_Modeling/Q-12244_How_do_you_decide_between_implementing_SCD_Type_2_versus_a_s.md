---
type: "practice"
difficulty: "Expert"
---

# How do you decide between implementing SCD Type 2 versus a separate mini-dimension when an attribute changes very frequently?

**Difficulty:** Expert · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ SCD Type 2 vs Mini-dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/SCD_Type_2_vs_Mini-dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing, deciding between using Slowly Changing Dimension (SCD) Type 2 and creating a separate mini-dimension when an attribute changes frequently depends on several factors including the business requirements, the impact of frequent changes, and the complexity you're willing to manage. Here's a step-by-step approach from first principles:

Understand SCD Type 2: SCD Type 2 involves keeping all historical versions of a record in the fact table by adding surrogate keys to track history.

#### Cons
ider the Frequency of Change: If changes are very frequent, using SCD Type 2 can lead to an explosion of records and complexity in queries.

#### Triaging the Question

The core of this question is about balancing data integrity with query performance when handling rapidly changing attributes. The interviewer likely wants to assess your understanding of both SCD Type 2 and mini-dimension approaches, as well as your ability to make a trade-off between them based on specific use cases.

#### First-Principles Solution
Identify the Attribute’s Change Frequency: If an attribute changes very frequently, it can create a large number of new records in the fact table.
Evaluate Query Performance Impact: Frequent updates and increased record count can slow down query performance, especially on complex joins involving many historical records.

#### Cons
ider Data Redundancy: Using mini-dimensions reduces data redundancy by keeping history in a separate dimension rather than spreading it across multiple rows in the fact table.
Detailed Reasoning

SCD Type 2:

#### Pros
Ensures referential integrity.
Maintains all historical versions of records, which is essential for audit trails and compliance requirements.
Facilitates easier rollbacks if something goes wrong.

#### Cons
Increased storage due to duplicated history.
Potential performance degradation with frequent changes.

Mini-Dimension Approach:

#### Pros
Reduces data redundancy in fact tables, potentially improving query performance.
Simplifies the schema by reducing the number of records and joins required for certain queries.

#### Cons
More complex to maintain because you need to ensure consistency between the mini-dimension and the main dimension.
Requires additional logic to handle changes correctly.

#### Example Scenario

#### Cons
ider a scenario where customer addresses frequently change, impacting sales data. If the address is updated multiple times in a short period, using SCD Type 2 might lead to many surrogate keys, making queries slower and more complex.

#### Implementation
Decision
Frequency of Change: If an attribute changes less than once per week, SCD Type 2 may be acceptable.
Query Performance: For attributes with frequent updates (e.g., addresses changing daily), a mini-dimension approach might be more efficient.
Business Requirements: Consider whether audit trails and historical data are necessary for compliance or business purposes.

#### Closing Summary

In summary, the choice between SCD Type 2 and mini-dimensions depends on balancing data integrity with query performance. Frequent attribute changes often necessitate a mini-dimension approach to maintain optimal query performance. However, this decision should be made after thorough consideration of all business and technical implications.

#### Common Mistakes/Follow-Ups
Overlooking Performance Impact: Not considering the impact of frequent updates on query performance.
Ignoring Business Requirements: Failing to weigh the importance of historical data versus current records in critical business processes.
Lack of Schema Design: Not understanding how changes will be handled and managed over time.
Triangulation

To ensure a robust solution, you might want to validate this decision with stakeholders and perhaps conduct a proof-of-concept before fully implementing the chosen approach.

**Comparison:**

    | SaleID | CustomerSK | AddressMiniDimID |
    | ------ | ---------- | ---------------- |
    | 100    | 1001       | 501              |
    | 200    | 1002       | 502              |
    | 300    | 1003       | 503              |
    | CustomerID | FirstName | LastName |
    | ---------- | --------- | -------- |
    | 1001       | John      | Doe      |
    | 1002       | Jane      | Smith    |
    | 1003       | Bob       | Johnson  |
    | AddressMiniDimID | CustomerSK | Street   | City  |
    | ---------------- | ---------- | -------- | ----- |
    | 501              | 1001       | OldStreet| OldCity|
    | 502              | 1002       | OldStreet| OldCity|
    | 503              | 1003       | OldStreet| OldCity|

**Code:**

```sql
-- Example SCD Type 2 Implementation in SQL

CREATE TABLE SalesFact (
    SaleID INT,
    CustomerSK INT,
    AddressMiniDimID INT,
    -- Other fact table columns
)

CREATE TABLE DimCustomer (
    CustomerID INT PRIMARY KEY,
    FirstName VARCHAR(50),
    LastName VARCHAR(50)
)

CREATE TABLE DimAddressMini (
    AddressMiniDimID INT PRIMARY KEY,
    CustomerSK INT FOREIGN KEY REFERENCES DimCustomer(CustomerID),
    Street VARCHAR(100),
    City VARCHAR(100),
    State CHAR(2),
    ZipCode VARCHAR(10)
)

-- Example SCD Type 2 Update Logic
UPDATE DimAddressMini
SET AddressMiniDimID = NewAddressMiniDimID,
    Street = 'NewStreet',
    City = 'NewCity'
WHERE CustomerSK = OldCustomerSK AND State = 'OldState'
```

**Example input / output:**

```text
flowchart TD;
    SaleFact[SalesFact Table] --> SCDType2[SCD Type 2] --> MiniDim[Mini-Dimension]
    SaleFact --> NoChange[No Change]
    SCDType2 -- High Performance --> QueryPerformance[Improved Query Performance]
    MiniDim -- Reduced Data Redundancy --> DataRedundancy[Reduced Data Redundancy]
```

**Data:**

```json
{
  "SalesFact": {
    "SaleID": [100, 200, 300],
    "CustomerSK": [1001, 1002, 1003],
    "AddressMiniDimID": [501, 502, 503]
  },
  "DimCustomer": {
    "CustomerID": [1001, 1002, 1003],
    "FirstName": ["John", "Jane", "Bob"]
  },
  "DimAddressMini": {
    "AddressMiniDimID": [501, 502, 503],
    "CustomerSK": [1001, 1002, 1003],
    "Street": ["OldStreet", "OldStreet", "OldStreet"],
    "City": ["OldCity", "OldCity", "OldCity"]
  }
}
```

**Flow:**

1. Yes
2. No
3. Yes
4. No
5. Attribute Changes VeryFrequently
6. Is Query PerformanceSuffering?
7. Consider Mini-Dimension
8. Do You Need Historical Data?
9. SCD Type 2
10. No Change Needed

