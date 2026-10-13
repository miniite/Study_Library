---
type: "practice"
difficulty: "Intermediate"
---

# How do you phrase a fact table grain as a single clear sentence? Give an example.

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1/2, SDE-2/3 · [↑ Fact Table Grain](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_Table_Grain.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

When modeling data, particularly in star or snowflake schemas, it's crucial to articulate the grain of a fact table succinctly and clearly. The grain defines the level at which measurements are captured. For instance, if we're dealing with sales data, the granularity should be specific enough that it uniquely identifies each transaction.

To phrase the fact table grain as a single clear sentence: "Each row in the fact table represents a unique transaction of a product by a customer on a specific date."

For example, consider a retail scenario. The fact table grain could be defined as: "Each row in the fact table captures a unique sale made to a specific customer for a particular product on a given day at a specified store." This sentence ensures that every record represents an individual transaction with all relevant attributes.

**Comparison:**

```
SaleID	CustomerID	ProductID	StoreID	SaleDate	QuantitySold	TotalAmount
12345	67890	23456	34567	2023-10-01	3	14.99
```

**Code:**

```sql
-- Example SQL Fact Table Definition
CREATE TABLE SalesFact (
    SaleID INT PRIMARY KEY,  -- Unique identifier for each sale
    CustomerID INT NOT NULL, -- Identifier for the customer who made the purchase
    ProductID INT NOT NULL,  -- Identifier for the product sold
    StoreID INT NOT NULL,    -- Identifier for the store where the sale occurred
    SaleDate DATE NOT NULL,  -- Date of the sale
    QuantitySold INT NOT NULL, -- Number of units sold in this transaction
    TotalAmount DECIMAL(10,2) NOT NULL -- Total amount paid for this sale
);
```

**Example input / output:**

```

Given the example input:

JSON
Copy
{
    "SaleID": 12345,
    "CustomerID": 67890,
    "ProductID": 23456,
    "StoreID": 34567,
    "SaleDate": "2023-10-01",
    "QuantitySold": 3,
    "TotalAmount": 14.99
}

The corresponding output (a row in the SalesFact table) would be:

JSON
Copy
{
    "SaleID": 12345,
    "CustomerID": 67890,
    "ProductID": 23456,
    "StoreID": 34567,
    "SaleDate": "2023-10-01",
    "QuantitySold": 3,
    "TotalAmount": 14.99
}
```

**Data:**

```json
{
    "SaleID": 12345,
    "CustomerID": 67890,
    "ProductID": 23456,
    "StoreID": 34567,
    "SaleDate": "2023-10-01",
    "QuantitySold": 3,
    "TotalAmount": 14.99
}
```

**Flow:**

1. Yes
2. No
3. Transaction Occurs
4. Is it a unique transaction?
5. Record Sale in Fact Table
6. Log as an Error or DuplicateCheck
7. Unique Transaction Recorded

