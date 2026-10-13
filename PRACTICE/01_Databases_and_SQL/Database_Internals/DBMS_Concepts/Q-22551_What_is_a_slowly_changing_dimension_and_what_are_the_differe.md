---
type: "practice"
difficulty: "Intermediate"
---

# What is a slowly changing dimension and what are the differences between Type 1 Type 2 and Type 3?

**Difficulty:** Intermediate · **Levels:** Fresher · [↑ Late Arriving Dimensions](<../../../../01_Databases_and_SQL/Database_Internals/DBMS_Concepts/Late_Arriving_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Slowly changing dimensions (SCDs) in data warehousing refer to attributes of entities that change over time, such as customer addresses or product descriptions. Understanding SCDs is crucial because they affect how historical changes are captured and managed in the database.

The question here is asking for a detailed explanation of three types of SCDs: Type 1, Type 2, and Type 3. Each type offers different strategies to handle attribute changes over time, impacting data consistency and integrity differently.

#### Triaging the Question
Type Probing: The interviewer likely wants you to understand how each type handles historical data.

#### Assumptions
We'll assume a typical ETL (Extract, Transform, Load) environment in a data warehouse.
First Principles: From basic principles of database management and data warehousing.
Detailed Explanation
Type 1: Overwrite the Old Value

In Type 1 SCDs, when an attribute changes, the old value is simply replaced with the new one. This approach does not preserve history but keeps the schema simple.

#### Example
If a customer's address changes, the new address overwrites the old one in the same row.
SQL
UPDATE Customers SET Address = 'New Address' WHERE CustomerID = 1;

#### Pros

Simple and straightforward to implement.
Efficient for queries since there are no historical versions.

#### Cons

Loss of historical data, making it difficult to track changes over time.
May lead to data anomalies if the schema is not carefully managed.
Type 2: Add a New Row with a Time Stamped Column

In Type 2 SCDs, when an attribute changes, a new row is added to the table with an additional timestamp. This preserves history and allows tracking of changes.

#### Example
If a customer's address changes, a new row is added with the new address and a timestamp.
SQL
INSERT INTO Customers (CustomerID, Name, Address, EffectiveDate, ExpireDate)
VALUES (1, 'John Doe', 'New Address', CURRENT_TIMESTAMP, NULL);

#### Pros

Retains historical data for detailed analysis.
Clear tracking of changes over time.

#### Cons

Requires more storage space as each change adds a new row.
Can complicate query performance if not optimized.
Type 3: Use a Junction Table

Type 3 SCDs involve using an additional junction table to maintain historical versions. This method is less common but can be very powerful for complex scenarios.

#### Example
A customer's address changes, and this change is recorded in a separate AddressHist table.
SQL
INSERT INTO AddressHistory (CustomerID, OldAddress, NewAddress, EffectiveDate)
VALUES (1, 'Old Address', 'New Address', CURRENT_TIMESTAMP);

#### Pros

Highly flexible for complex historical tracking.
Can handle multiple changes and attributes more granularly.

#### Cons

More complex to implement and manage.
Requires careful schema design to ensure consistency and performance.

#### Closing Summary

Type 1, Type 2, and Type 3 SCDs offer different trade-offs in terms of storage, query complexity, and historical data retention. Understanding these differences is crucial for designing robust data warehousing solutions that meet specific business needs.

#### Common Mistakes/Follow-Ups
Mistake: Overlooking the importance of Type 2 SCDs for preserving history.

#### Follow-Up
How would you choose between Type 1 and Type 2 based on project requirements?
Further Discussion: Discuss scenarios where Type 3 might be necessary.

**Comparison:**

```
CustomerID	Name	Address	EffectiveDate	ExpireDate
1	John	Old Address	2023-10-01	NULL
1	John	New Address	2023-10-05	NULL
```

**Flow:**

1. No
2. Yes
3. No
4. Yes
5. Type 1: Overwrite
6. Is Historical Analysis Required?
7. Use Type 1
8. Type 2: Add New Row withTimestamp
9. Type 3: Junction Table
10. Complexity Allowed?
11. Use Type 2
12. Use Type 3

