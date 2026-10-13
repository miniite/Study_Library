---
type: "practice"
difficulty: "Intermediate"
---

# Describe how you would design a role-playing date dimension so order-date, ship-date, and return-date reports all work without duplicating the dimension.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Date_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To design a role-playing date dimension that supports order-date, ship-date, and return-date reports without duplicating the dimension, we need to create a flexible and scalable data model. This involves understanding the requirements, designing the schema, and ensuring that our implementation can handle various reporting needs efficiently.

The key idea is to leverage a star or snowflake schema while maintaining a single date dimension table that serves multiple purposes. We'll use a combination of fact tables and date dimensions to achieve this without redundancy.

#### Triaging the Question
Understanding Requirements: The primary goal is to support reports for order-date, ship-date, and return-date scenarios.
Designing the Schema: We need to ensure that our design can handle different types of dates in a single, efficient date dimension table.
Handling Reporting Scenarios: Different fact tables will reference this single date dimension based on their specific needs.
Step-by-Step Solution

Define the Date Dimension:

Create a DimDate table with columns for various levels such as year, month, day, etc., along with surrogate keys.

Create Fact Tables:

FactOrders: This will capture all order-related details and reference the DimDate based on the order date.
FactShipments: This will capture shipment-related data and reference the DimDate based on the ship date.
FactReturns: This will capture return-related data and reference the DimDate based on the return date.

Use of Surrogate Keys:

Ensure that each fact table has a surrogate key to uniquely identify each record, which is independent of the date dimension.

Star Schema Design:

The star schema will consist of a single DimDate table at the center, with multiple FactOrders, FactShipments, and FactReturns tables branching out from it.

Reporting Flexibility:

For order-date reports, join FactOrders with DimDate.
For ship-date reports, join FactShipments with DimDate.
For return-date reports, join FactReturns with DimDate.

#### Example
Schema
SQL
CREATE TABLE DimDate (
    DateID INT PRIMARY KEY,
    Year SMALLINT NOT NULL,
    Month SMALLINT NOT NULL,
    Day SMALLINT NOT NULL,
    -- Add other necessary date attributes
);

CREATE TABLE FactOrders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    DateID INT NOT NULL REFERENCES DimDate(DateID),
    OrderAmount DECIMAL(10,2) NOT NULL,
    -- Other order-related columns
);

CREATE TABLE FactShipments (
    ShipmentID INT PRIMARY KEY,
    SupplierID INT NOT NULL,
    DateID INT NOT NULL REFERENCES DimDate(DateID),
    Quantity INT NOT NULL,
    -- Other shipment-related columns
);

CREATE TABLE FactReturns (
    ReturnID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    DateID INT NOT NULL REFERENCES DimDate(DateID),
    RefundAmount DECIMAL(10,2) NOT NULL,
    -- Other return-related columns
);

#### Closing Summary
and Common Mistakes/Follow-Ups
Summary: By using a single DimDate table with multiple fact tables, we can efficiently support different date-based reports without redundancy. This approach is scalable and maintains data integrity.

#### Common Mistakes
Duplicating the date dimension in each fact table, leading to redundancy.
Failing to properly join fact tables with the DimDate table, causing incorrect reporting results.

#### Follow-Ups
How would you handle different time zones for the dates?
What about historical data and rolling updates on the DimDate table?
How can we optimize this design for performance in a large-scale environment?

**Comparison:**

| DateID | Year | Month | Day |
|--------|------|-------|-----|
| 1      | 2023 | 5     | 1   |
| OrderID | CustomerID | DateID | OrderAmount |
|---------|------------|--------|-------------|
| 1001    | 1          | 1      | 150.00      |
| ShipmentID | SupplierID | DateID | Quantity |
|------------|------------|--------|----------|
| 2001       | 1          | 1      | 30       |
| ReturnID | CustomerID | DateID | RefundAmount |
|----------|------------|--------|--------------|
| 3001     | 1          | 1      | 50.00        |

**Code:**

```sql
CREATE TABLE DimDate (
    DateID INT PRIMARY KEY,
    Year SMALLINT NOT NULL,
    Month SMALLINT NOT NULL,
    Day SMALLINT NOT NULL,
    -- Add other necessary date attributes
);

CREATE TABLE FactOrders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    DateID INT NOT NULL REFERENCES DimDate(DateID),
    OrderAmount DECIMAL(10,2) NOT NULL,
    -- Other order-related columns
);

CREATE TABLE FactShipments (
    ShipmentID INT PRIMARY KEY,
    SupplierID INT NOT NULL,
    DateID INT NOT NULL REFERENCES DimDate(DateID),
    Quantity INT NOT NULL,
    -- Other shipment-related columns
);

CREATE TABLE FactReturns (
    ReturnID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    DateID INT NOT NULL REFERENCES DimDate(DateID),
    RefundAmount DECIMAL(10,2) NOT NULL,
    -- Other return-related columns
);
```

**Example input / output:**

```text
Order-date report input:
SELECT DimDate.Year, DimDate.Month, SUM(FactOrders.OrderAmount)
FROM FactOrders
JOIN DimDate ON FactOrders.DateID = DimDate.DateID
WHERE DimDate.Year = 2023 AND DimDate.Month = 5

Output:
Year | Month | TotalSales
2023 | 5     | 150.00

Ship-date report input:
SELECT DimDate.Year, DimDate.Month, SUM(FactShipments.Quantity)
FROM FactShipments
JOIN DimDate ON FactShipments.DateID = DimDate.DateID
WHERE DimDate.Year = 2023 AND DimDate.Month = 5

Output:
Year | Month | TotalQuantityShipped
2023 | 5     | 30.00

Return-date report input:
SELECT DimDate.Year, DimDate.Month, SUM(FactReturns.RefundAmount)
FROM FactReturns
JOIN DimDate ON FactReturns.DateID = DimDate.DateID
WHERE DimDate.Year = 2023 AND DimDate.Month = 5

Output:
Year | Month | TotalRefunds
2023 | 5     | 50.00
```

**Data:**

```plaintext
DimDate:
- DateID: 1
- Year: 2023
- Month: 5
- Day: 1

FactOrders:
- OrderID: 1001
- CustomerID: 1
- DateID: 1
- OrderAmount: 150.00

FactShipments:
- ShipmentID: 2001
- SupplierID: 1
- DateID: 1
- Quantity: 30

FactReturns:
- ReturnID: 3001
- CustomerID: 1
- DateID: 1
- RefundAmount: 50.00
```

**Flow:**

1. DimDate
2. Order ID
3. Shipment ID
4. Return ID
5. Year, Month, TotalRefunds

