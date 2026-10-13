---
type: "practice"
difficulty: "Basic"
---

# What is a star schema and why is it called a star?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Star Schemas and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Star_Schemas_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A star schema is a specific type of data model used in dimensional databases, where data is organized into a central fact table surrounded by multiple dimension tables. The structure resembles a star with rays emanating from its center, hence the name "star schema." Let's dive deeper into why it's called this and explore its key features.

**Comparison:**

```
Fact Table	Dimension Table (Product)	Dimension Table (Store)
SalesID, DateID, ProductID, StoreID, Quantity, Amount	ProductID, ProductName, Category	StoreID, StoreName
```

**Code:**

```
Fact Table
DIM_Day
DIM_Product
DIM_Store

This Mermaid diagram illustrates the basic structure of a star schema, where the fact table (A) is connected to multiple dimension tables (B1, B2, and B3).
```

**Example input / output:**

```

For a star schema, consider the following scenario where we are querying sales data:

Input Query:

SQL
Copy
SELECT
    D.ProductName,
    S.StoreName,
    SUM(F.Quantity) as TotalSales,
    SUM(F.Amount) as TotalAmount
FROM FactTable F
JOIN DimProduct D ON F.ProductID = D.ProductID
JOIN DimStore S ON F.StoreID = S.StoreID
WHERE F.DateID BETWEEN 20230401 AND 20230430
GROUP BY D.ProductName, S.StoreName;

Output:

JSON
Copy
[
    {
        "ProductName": "Coffee",
        "StoreName": "Corner Store",
        "TotalSales": 5,
        "TotalAmount": 25.99
    },
    {
        "ProductName": "Tea",
        "StoreName": "Town Center",
        "TotalSales": 3,
        "TotalAmount": 18.00
    }
]
```

**Data:**

```
Fact Table Example:
JSON
Copy
{
    "SalesID": 1,
    "ProductID": 101,
    "StoreID": 507,
    "DateID": 20230401,
    "Quantity": 5,
    "Amount": 25.99
}
Dimension Table Example:
JSON
Copy
{
    "ProductID": 101,
    "ProductName": "Coffee",
    "Category": "Drinks"
}
```

