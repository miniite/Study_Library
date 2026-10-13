---
type: "practice"
difficulty: "Deep"
---

# When would you use SCD Type 3 instead of Type 2?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ SCD Type 1 vs Type 2](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To address when to use SCD Type 3 over SCD Type 2 in a data warehousing context, let's first summarize and then delve into the nuances.

TL;DR

SCD (Slowly Changing Dimension) Type 3 is used when you need historical information for each record but also want to maintain the original records without creating new surrogate keys. It’s particularly useful when you have complex business requirements that demand keeping a detailed history of changes over time.

#### Triaging the Question

The question is about choosing between SCD Type 2 and SCD Type 3 in data warehousing. The key elements being probed are:

Understanding of SCD types (Type 1, Type 2, Type 3)
Business requirements analysis

#### Trade-offs
between different types
First Principles
Background on SCD Types
SCD Type 1: Overwrite the existing record with new values.
SCD Type 2: Create a new surrogate key for each change and keep old records unchanged.
SCD Type 3: Keep both old and current version of the record, using additional columns to track changes.

#### Solution
When to Use SCD Type 3
Complex Historical Tracking Requirements:
If your business needs detailed historical information about all versions of a dimension, you need SCD Type 3.
Maintaining Data Consistency Across Time:
To ensure that data is not lost or altered, and to maintain the integrity of past records.

#### Example Scenario

Imagine a retail company where products have different specifications over time (e.g., color, size). You might want to track all historical versions of these products without losing their original entries. Here’s how you would implement it:

SQL
CREATE TABLE Products (
    ProductID INT PRIMARY KEY,
    Name VARCHAR(255),
    Color VARCHAR(50),
    Size VARCHAR(10),
    EffectiveDate DATE NOT NULL,
    EndDate DATE,
    Active BOOLEAN DEFAULT TRUE
);

In this schema, EffectiveDate and EndDate are used to track when a product record becomes effective or ceases to be active.

Step-by-Step Implementation
Insert the New Record:
Insert a new row with the updated values.
Set Effective Date:
Set the EffectiveDate of the new record to the current date and time.
Update End Date for Previous Records:
Set the EndDate of any previous records that are no longer valid.

#### Closing Summary

SCD Type 3 is ideal when you need a comprehensive history of changes in dimension data, ensuring historical integrity without altering original records. It provides flexibility and detailed tracking, which can be crucial for complex business scenarios requiring robust historical data management.

#### Common Mistakes/Follow-Ups
Misunderstanding Business Requirements: Failing to fully comprehend the business needs might lead to incorrect implementation.

#### Performance Considerations
While SCD Type 3 provides detailed history, it can impact query performance due to additional columns and records.
Data Quality and Integrity: Ensuring that data is consistently recorded and updated can be challenging, especially in large datasets.

#### Example
Code
SQL
-- Inserting a new record with effective date
INSERT INTO Products (Name, Color, Size, EffectiveDate)
VALUES ('Sneaker', 'Black', 'M', CURRENT_DATE);

-- Updating the end date for previous records
UPDATE Products
SET EndDate = CURRENT_DATE - INTERVAL 1 DAY,
    Active = FALSE
WHERE ProductID IN (SELECT ProductID FROM Products WHERE Name = 'Sneaker' AND Color = 'Black' AND Size = 'M'
                    AND EffectiveDate < CURRENT_DATE);

