---
type: "practice"
difficulty: "Deep"
---

# What is a role-playing dimension? How would a single date dimension play the role of order date and ship date?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Date Dimensions](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Date_Dimensions.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

A role-playing dimension in data warehousing refers to using the same fact table with multiple versions of the same dimension, each serving different perspectives or roles. In the context of a retail database, the single "Date" dimension can play the role of both "Order Date" and "Ship Date." This concept helps in providing flexibility and reducing redundancy.

To understand this better, let's break down the problem from first principles.

**Comparison:**

| OrderID | OrderDate  | ShipDate | Revenue | Quantity |
| 1       | 2023-04-05 | 2023-04-10| 100.00  | 2        |

**Code:**

```
MERMAID
Copy
graph LR;
    A[Fact Table] --> B(Order Date Dimension)
    A --> C(Ship Date Dimension)
    B --> D[Order Facts]
    C --> E[Ship Facts]
```

**Example input / output:**

```

Consider the input data:

PLAINTEXT
Copy
OrderID | ShipDate       | Revenue | Quantity
--------|---------------|---------|---------
 1      | 2023-04-05    | 100.00  | 2
 2      | 2023-04-10    | 75.00   | 1

The Sales_Fact table contains both order and ship dates for each transaction, allowing us to analyze them separately.
```

**Data:**

```

No specific data schemas or JSON are required for this explanation, but let's consider a simplified fact table schema.

SQL
Copy
CREATE TABLE Sales_Fact (
  OrderID INT,
  ShipDate DATE,
  Revenue DECIMAL(10,2),
  Quantity INT
);
```

**Flow:**

1. Fact Table
2. Order Date Role
3. Ship Date Role
4. Orders Analysis
5. Shipment Analysis

