---
type: "practice"
difficulty: "Deep"
---

# What is a conformed dimension and why is it useful across multiple data marts?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ SCD Type 1 vs Type 2](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Data_Warehousing/SCD_Type_1_vs_Type_2.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A conformed dimension (CD) in data warehousing is a dimension table that is shared across multiple data marts, ensuring consistency and reducing redundancy in your data warehouse schema. This concept is crucial because it allows different parts of an organization to query the same standardized data, enhancing accuracy and maintaining integrity.

To understand why conformed dimensions are useful, we need to start by understanding their purpose and how they function within a data warehousing environment.

**Comparison:**

```
Customer Dimension Table
dimension_id
1001
Sales Mart Fact Table
sales_id
2001
Marketing Mart Fact Table
marketing_id
4001
```

**Code:**

```
MERMAID
Copy
graph TD;
    A[Data Warehouse] --> B[Data Mart 1];
    A --> C[Data Mart 2];
    B --> D{Conformed Dimension};
    C --> D;
```

**Example input / output:**

```

Let's consider a simple example where we have two data marts, Sales and Marketing, both using the same conformed dimension Customer.

PLAINTEXT
Copy
Input:
Sales Mart Data:
- CustomerID: 1001, DimName: "John Doe", DimDescription: "Customer from Sales"
Marketing Mart Data:
- CustomerID: 1001, DimName: "John Doe", DimDescription: "Customer from Marketing"

Output (after conformed dimension is applied):
Sales Mart Result:
- CustomerID: 1001, DimName: "John Doe", DimDescription: "Customer from Sales"
Marketing Mart Result:
- CustomerID: 1001, DimName: "John Doe", DimDescription: "Customer from Marketing"

Both marts are using the same `Customer` record.
```

**Data:**

```

A conformed dimension schema might look like this:

JSON
Copy
{
  "dimension_id": "string",
  "dim_name": "string",
  "dim_description": "string",
  "dim_valid_from": "datetime",
  "dim_valid_to": "datetime"
}
```

**Flow:**

1. Data Source
2. Join with ConformedDimension
3. Standardized Data
4. Query from Multiple Marts
5. Unified View of Customer Data

