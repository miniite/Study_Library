---
type: "practice"
difficulty: "Deep"
---

# What is a slowly changing dimension Type 4 (history table) and how does it differ from Type 2?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ SCD Type 1 vs Type 2](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Summary

A slowly changing dimension Type 4, also known as a history table, is an advanced strategy used in data warehousing to manage changes over time while maintaining historical records. It differs from Type 2 (full history) by creating a new record for each change instead of updating the existing one. This approach allows for detailed analysis but requires careful handling to avoid redundancy and maintain performance.

#### Triaging the Question

The question asks you to explain two specific types of slowly changing dimensions, which are part of the SCD (Slowly Changing Dimension) strategy in data warehousing. The interviewer is likely probing your understanding of how different methods handle changes over time and their implications on data integrity and query performance. Key assumptions include familiarity with dimensional modeling and the need for historical tracking.

From First Principles
Type 2: Full History

In Type 2, when a dimension attribute changes, you create a new record in the dimension table to preserve the history. This approach is straightforward but can lead to data bloat over time, especially if many changes occur frequently. Each change results in a new row, making it easier to track historical values but less efficient in terms of storage and query performance.

Type 4: History Table

Type 4 involves maintaining a separate history table that tracks all changes to the dimension attributes over time. A unique identifier (e.g., surrogate key) is used as a foreign key to link back to the current record in the dimension table. This method allows for precise historical analysis without bloating the main dimension table.

Detailed Solution

Structure and Design:

Dimension Table: Stores the most recent version of each dimension attribute.
History Table: Stores all previous versions of dimension attributes, using a surrogate key to link back to the current record in the dimension table.

Data Flow:

When an attribute changes, a new row is added to both the history table and the main dimension table. The foreign key in the history table points back to the current record in the dimension table.

Query Considerations:

For historical analysis, you can join the dimension table with its corresponding history table using the surrogate key.
Query performance is improved as only recent data is stored in the main dimension table, but this requires careful design and indexing strategies to avoid performance bottlenecks during joins.

#### Trade-offs

Advantages: Detailed historical tracking without bloating the main dimension table.
Disadvantages: Increased complexity due to maintaining a separate history table, potential for redundant data if not managed properly, and increased query processing time due to join operations.

#### Closing Summary

Type 4 (history table) is an effective method in slowly changing dimensions when detailed historical tracking is necessary without compromising the performance of your main dimension table. It requires careful design to manage complexity and ensure efficient queries while maintaining data integrity.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the need for proper indexing, which can lead to poor query performance.

#### Follow-Up
How would you handle high-cardinality attributes in Type 4? (Discuss partitioning strategies or using a denormalized approach.)
Triaging the Follow-Up

The follow-up question aims to test your understanding of how to optimize Type 4 implementations for large datasets. Discussing partitioning, indexing, and denormalization techniques will demonstrate your depth of knowledge.

**Comparison:**

| CustomerID | FirstName | LastName | ChangeDate        | EffectiveFrom       | EffectiveTo      |
|-----------|----------|---------|------------------|--------------------|-----------------|
| 1         | John      | Doe     | 2023-10-04T12:00Z| 2023-10-01T00:00Z   | NULL             |
| 1         | Jane      | Doe     | 2023-10-05T13:00Z| 2023-10-04T12:00Z   | 9999-12-31T23:59Z|

**Code:**

```sql
-- Example of creating a history table in SQL

CREATE TABLE DimCustomerHistory (
    CustomerID INT,
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    ChangeDate TIMESTAMP,
    EffectiveFrom TIMESTAMP,
    EffectiveTo TIMESTAMP,
    PRIMARY KEY (CustomerID, ChangeDate)
);

INSERT INTO DimCustomerHistory
SELECT 1, 'John', 'Doe', CURRENT_TIMESTAMP - INTERVAL '3 days', CURRENT_TIMESTAMP - INTERVAL '4 days', NULL;

INSERT INTO DimCustomerHistory
SELECT 1, 'Jane', 'Doe', CURRENT_TIMESTAMP, CURRENT_TIMESTAMP - INTERVAL '3 days', CURRENT_TIMESTAMP;
```

**Example input / output:**

```text
Input:
- New customer record with ID 1, FirstName = 'John', LastName = 'Doe'
- Change to customer record with ID 1, FirstName = 'Jane', LastName = 'Doe'

Output (history table):
CustomerID | FirstName | LastName | ChangeDate        | EffectiveFrom       | EffectiveTo
---------------------------------------------------------------
1          | John      | Doe      | 2023-10-04T12:00Z | 2023-10-01T00:00Z   | NULL
1          | Jane      | Doe      | 2023-10-05T13:00Z | 2023-10-04T12:00Z   | 2099-12-31T23:59Z
```

**Data:**

```json
{
    "DimCustomer": {
        "CustomerID": 1,
        "FirstName": "John",
        "LastName": "Doe"
    },
    "DimCustomerHistory": [
        {"CustomerID": 1, "FirstName": "John", "LastName": "Doe", "ChangeDate": "2023-10-01T00:00:00Z"},
        {"CustomerID": 1, "FirstName": "Jane", "LastName": "Doe", "ChangeDate": "2023-10-04T00:00:00Z"}
    ]
}
```

**Flow:**

1. No
2. Yes
3. Customer ID
4. Change in Attribute?
5. Use Current Record
6. Create New History Row
7. Link to Current Record
8. Store Attribute Change

