---
type: "practice"
difficulty: "Deep"
---

# How do filtered indexes work, and give a production scenario where a filtered index dramatically reduced index size and improved a query.

**Difficulty:** Deep · **Levels:** SDE-1, SDE-1/2 · [↑ Clustered Indexes and Indexed Views](<../../../../TOOLS/SQL_Server/SQL_Server_Platform/Clustered_Indexes_and_Indexed_Views.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Filtered indexes in SQL Server are designed to store only rows that meet a specific condition. This can lead to significant space savings and performance improvements, particularly when querying data with frequent filters or WHERE clauses. By reducing the size of the index, it also speeds up operations like seeks and scans.

#### Triaging the Question

The core of this question is understanding how filtered indexes function, their benefits, and a practical example where they can be highly effective. The key points to cover include:

How a filtered index is created and what conditions it enforces.
The impact on storage space and query performance.
Common scenarios where filtered indexes are useful.

#### First Principles Solution

Filtered indexes are created using the WITH (FILTERED = <expression>) clause in SQL Server. For instance, if you have a table with frequent queries that filter by a specific column value, creating an index on only those rows can drastically reduce its size and improve query performance. Let's consider a scenario where we track product sales data.

Scenario: E-commerce Product Sales Data

Imagine an e-commerce platform tracking daily sales of various products across different regions. The Sales table has columns like ProductID, Region, Date, and UnitsSold. A common query pattern is to retrieve the total units sold for a specific product in a particular region.

SQL
CREATE TABLE Sales (
    ProductID INT,
    Region NVARCHAR(50),
    Date DATE,
    UnitsSold INT,
    -- other columns...
);

-- Example data insertion
INSERT INTO Sales (ProductID, Region, Date, UnitsSold)
VALUES (1, 'North', '2023-10-01', 100),
       (1, 'South', '2023-10-01', 50),
       (2, 'East', '2023-10-01', 75),
       -- more data...
Creating a Filtered Index

To improve query performance for products in the "North" region, we can create a filtered index on the UnitsSold column only for rows where Region = 'North'.

SQL
CREATE NONCLUSTERED INDEX IX_Sales_North ON Sales (UnitsSold)
WHERE Region = 'North';

This creates an index that includes only those rows from the Sales table where Region = 'North'. The resulting index will be much smaller than a non-filtered index, reducing storage space and improving query performance for relevant queries.

Query Performance

#### Cons
ider the following query:

SQL
SELECT SUM(UnitsSold) AS TotalUnits
FROM Sales
WHERE Region = 'North';

Without the filtered index, SQL Server would need to scan or seek through all rows in the Sales table. With the filtered index, the engine can directly perform a seek operation on the smaller, more targeted index.

Impact Analysis
Storage Space: The filtered index will store only the relevant rows, significantly reducing the space needed compared to an unfiltered index.
Query Performance: Queries that filter by Region = 'North' will benefit from this index, as the engine can perform a seek operation on the smaller index.

#### Closing Summary
and Common Mistakes

Creating filtered indexes is a powerful technique for optimizing performance in specific scenarios. However, they should be used judiciously:

Ensure the condition you filter by (e.g., Region = 'North') matches your query patterns.
Filtered indexes can only be beneficial if they reduce the index size significantly and the queries are frequently run with the same filter.

#### Common mistakes
include using filters that do not significantly reduce the index size or applying them to tables where the majority of queries do not use the filtering condition. Always validate the impact before implementing a filtered index.

Common Follow-Ups
How does SQL Server decide when to create an index?
Are there any trade-offs or considerations when using filtered indexes in production?

**Comparison:**

```
ProductID	Region	Date	UnitsSold
1	North	2023-10-01	100
1	South	2023-10-01	50
2	East	2023-10-01	75
```

**Code:**

```sql
-- Example table creation and data insertion
CREATE TABLE Sales (
    ProductID INT,
    Region NVARCHAR(50),
    Date DATE,
    UnitsSold INT
);

INSERT INTO Sales (ProductID, Region, Date, UnitsSold)
VALUES (1, 'North', '2023-10-01', 100),
       (1, 'South', '2023-10-01', 50),
       (2, 'East', '2023-10-01', 75);

-- Creating a filtered index
CREATE NONCLUSTERED INDEX IX_Sales_North ON Sales (UnitsSold)
WHERE Region = 'North';
```

**Example input / output:**

```text
-- Input query without filtered index
SELECT SUM(UnitsSold) AS TotalUnits
FROM Sales
WHERE Region = 'North';

-- Output (assuming the filtered index is used)
TotalUnits: 100

-- Explanation of output
This example shows that using a filtered index can significantly improve performance and storage efficiency for specific queries.
```

**Data:**

```json
{
    "Sales": [
        {"ProductID": 1, "Region": "North", "Date": "2023-10-01", "UnitsSold": 100},
        {"ProductID": 1, "Region": "South", "Date": "2023-10-01", "UnitsSold": 50},
        {"ProductID": 2, "Region": "East", "Date": "2023-10-01", "UnitsSold": 75}
    ]
}
```

**Flow:**

1. Sales Table
2. Region = 'North'
3. Filtered Index
4. Other Regions
5. Non-filtered Index
6. Smaller Index Size & FasterQueries
7. Fewer Rows to Scan

