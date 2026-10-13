---
type: "practice"
difficulty: "Intermediate"
---

# In a star schema, why do you usually index the foreign keys in the fact table that point to each dimension?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Join Algorithm Selection](<../../../../01_Databases_and_SQL/Query_Performance/Indexing/Join_Algorithm_Selection.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

#### Overview

In a star schema, indexing foreign keys in the fact table is crucial because it significantly speeds up query performance by reducing the number of rows that need to be scanned. This optimization leverages the structure where a single fact table is associated with multiple dimension tables, often leading to complex queries involving joins.

#### Triage

The question asks why foreign keys are typically indexed in the fact table within a star schema. The core idea here is understanding how indexing these keys affects query performance and overall database efficiency. We need to assume that we're working with a standard star schema design used for data warehousing, where the fact table has many rows but relatively few columns (dimensions), and each dimension can have millions of records.

Step-by-Step Solution:

Understanding the Star Schema:

A star schema consists of a central fact table and several dimension tables.
The fact table contains measurable facts and foreign keys to dimensions.
Dimension tables store detailed attributes (e.g., dates, products).

Query Performance Considerations:

Queries often involve joining the fact table with multiple dimension tables based on these foreign keys.
Without proper indexing, the database engine must perform a full table scan for each join operation.

Importance of Indexing Foreign Keys:

Indexes allow for faster lookup times in the fact table when performing joins.
By creating indexes on the foreign keys, we can quickly locate relevant rows in the dimension tables, thereby reducing the number of scans needed in the fact table.
For example, if a query involves filtering by date or product ID, an index on these fields will speed up the join operations.

Detailed Example:

#### Cons
ider a retail application with a fact table "Sales" and dimensions for "Date," "Product," and "Store."
A typical query might look like:
SQL
SELECT SUM(SalesAmount)
FROM Sales
JOIN Date ON Sales.DateID = Date.DateID
JOIN Product ON Sales.ProductID = Product.ProductID
WHERE Date.Year > 2019 AND Product.Category = 'Electronics';
Without indexes, this query would scan the entire "Sales" table and then join it with both dimension tables. With proper indexing, the database engine can quickly locate relevant rows in the "Date" and "Product" dimensions based on the filter conditions.

Cost Analysis:

Indexing foreign keys adds some overhead during insertions and updates but significantly improves read performance.
For high-frequency analytical queries, this trade-off is generally justified to achieve faster response times.

#### Conclusion

Indexing foreign keys in the fact table of a star schema optimizes query performance by facilitating quicker join operations and reducing the number of rows that need to be scanned.
This practice is essential for handling large datasets typical in data warehousing applications.

#### Common Mistakes
and Follow-Ups:
Mistake: Ignoring the importance of indexing can lead to slow query performance, especially with complex queries involving multiple joins.

#### Follow-Up
How would you handle the situation where the fact table has millions of rows? What about scenarios with frequent updates?

**Comparison:**

| Fact Table Schema | Dimension Table Schema |
|-------------------|-----------------------|
| SaleID (PK)       | DateID (PK)           |
| DateID (FK)       | Year                  |
| ProductID (FK)    | Category              |
| StoreID (FK)      | Name                  |
| SalesAmount       | Location              |

**Code:**

```sql
-- Example SQL for creating an index on a foreign key in the fact table
CREATE INDEX idx_sales_date_id ON Sales(DateID);
CREATE INDEX idx_sales_product_id ON Sales(ProductID);
```

**Example input / output:**

```

Example input:

SQL
Copy
-- Query without index
SELECT SUM(SalesAmount)
FROM Sales
JOIN Date ON Sales.DateID = Date.DateID
JOIN Product ON Sales.ProductID = Product.ProductID
WHERE Date.Year > 2019 AND Product.Category = 'Electronics';

-- Query with index
SELECT SUM(SalesAmount)
FROM Sales
JOIN Date ON Sales.DateID = Date.DateID
JOIN Product ON Sales.ProductID = Product.ProductID
WHERE Date.Year > 2019 AND Product.Category = 'Electronics';

Output:

Without index: Slow response time, full table scan.
With index: Faster response time, targeted row lookup.
```

**Data:**

```
Sample schema:
PLAINTEXT
Copy
Fact Table: Sales
  - SaleID (PK)
  - DateID (FK to DimDate)
  - ProductID (FK to DimProduct)
  - StoreID (FK to DimStore)
  - SalesAmount

Dimension Tables:
  - DimDate
    - DateID (PK)
    - Year
    - Month
    - Day
  - DimProduct
    - ProductID (PK)
    - Category
    - Name
  - DimStore
    - StoreID (PK)
    - Location
```

**Flow:**

1. Yes
2. No
3. Fact Table
4. Index on DateID
5. Quick Join with Date
6. Full Table Scan
7. Fast Query Execution
8. Slow Query Execution

