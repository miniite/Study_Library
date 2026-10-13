---
type: "practice"
difficulty: "Basic"
---

# What is a natural key versus a surrogate key in a dimension table, and how does the choice affect the primary key index?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Random Keys and Index Locality](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Random_Keys_and_Index_Locality.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In a database context, understanding the difference between a natural key and a surrogate key is crucial for effective query optimization and indexing design. A natural key refers to a unique identifier that exists in the real world and is used directly within the data model, whereas a surrogate key is an artificial identifier created specifically for database purposes.

The choice of primary key index can significantly impact performance, especially in dimension tables where frequent joins are common. Let's break down each concept and explore how their use affects indexing.

Natural Key vs Surrogate Key: Overview
Natural Key: A natural key directly uses a unique piece of data that exists naturally within the business context, such as a customer’s SSN or an employee’s ID number.
Surrogate Key: A surrogate key is a synthetic identifier with no inherent meaning in the business domain. It is often used to avoid data redundancy and improve performance.

#### Triaging the Question

The question probes your understanding of:

What natural and surrogate keys are.
Why we use them.
How their choice influences indexing decisions.

#### Assumptions
We assume a dimension table context, which is common in data warehousing scenarios.
The focus is on primary key selection rather than composite or foreign keys.

#### First-Principles Solution

Natural Key Usage:

#### Pros
Directly linked to real-world entities; easier for end-users who are familiar with the business domain.

#### Cons
May lead to data redundancy and potential performance issues due to large, variable-length keys (e.g., SSN).

Surrogate Key Usage:

#### Pros
Smaller, fixed-length key which improves join performance; avoids data redundancy; easier to manage and maintain.

#### Cons
Users may find it less intuitive as it lacks inherent meaning in the business context.

#### Example Scenario

#### Cons
ider a customer dimension table:

MARKDOWN
| CustomerID | FirstName | LastName | DOB       |
|------------|-----------|----------|-----------|
| 12345      | John      | Doe      | 1980-01-01|
| 67890      | Jane      | Smith    | 1975-03-15|

In this case, CustomerID is a surrogate key because it's an artificial identifier. If we used the customer’s SSN as the natural key:

MARKDOWN
| CustomerSSN   | FirstName | LastName | DOB       |
|---------------|-----------|----------|-----------|
| 123-45-6789   | John      | Doe      | 1980-01-01|
| 234-56-7890   | Jane      | Smith    | 1975-03-15|
Impact on Indexing

Surrogate Key:

Typically, a B-tree index is created on the surrogate key for fast lookups.

#### Example
CREATE INDEX idx_customer ON customer_dimension (CustomerID);

Natural Key:

If used, it might require a more complex indexing strategy due to its variability and length.

#### Example
CREATE INDEX idx_customer_ssn ON customer_dimension (CustomerSSN).

#### Closing Summary

Choosing between natural and surrogate keys is critical for database design. Surrogate keys are generally preferred in data warehousing environments due to their performance benefits, especially when dealing with frequent joins. However, the choice should also consider user-friendliness and compliance requirements.

#### Common Mistakes/Follow-Ups
Mistake: Using a natural key as the primary key without considering its length or variability.

#### Follow-Up
Discussing scenarios where both keys might be used together (e.g., using SSN for internal processing and CustomerID for external reporting).

**Comparison:**

```
CustomerID	FirstName	LastName	DOB
12345	John	Doe	1980-01-01
67890	Jane	Smith	1975-03-15
```

**Code:**

```sql
-- Example SQL for creating an index on a surrogate key

CREATE INDEX idx_customer ON customer_dimension (CustomerID);
```

