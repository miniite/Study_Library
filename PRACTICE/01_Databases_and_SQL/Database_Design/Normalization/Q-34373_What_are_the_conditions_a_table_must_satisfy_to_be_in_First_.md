---
type: "practice"
difficulty: "Basic"
---

# What are the conditions a table must satisfy to be in First Normal Form?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ First Normal Form](<../../../../01_Databases_and_SQL/Database_Design/Normalization/First_Normal_Form.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

First Normal Form (1NF) is the foundational level of database normalization, ensuring that each column in a table contains atomic (indivisible) values and there are no duplicate rows. To satisfy 1NF, a table must meet two key conditions:

Atomic Columns: Each value in every column should be an indivisible unit.
Unique Rows: No two rows within the same table can have identical values across all columns.

This ensures that each row represents a unique entity or fact and each column contains distinct information, making data manipulation simpler and more reliable.

**Code:**

```sql
-- Example schema to illustrate 1NF conditions
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT NOT NULL,
    ProductID INT NOT NULL,
    Quantity INT NOT NULL,
    Price DECIMAL(10,2) NOT NULL
);

-- Inserting data into the table (satisfies 1NF)
INSERT INTO Orders (OrderID, CustomerID, ProductID, Quantity, Price)
VALUES
(1, 101, 456, 3, 9.99),
(2, 102, 789, 2, 14.99);
```

**Example input / output:**

```

Input:

SQL
Copy
-- Inserting a row into Orders table
INSERT INTO Orders (OrderID, CustomerID, ProductID, Quantity, Price)
VALUES (1, 101, 456, 3, 9.99);

Output: Row successfully inserted.
```

**Data:**

```json
{
    "OrderID": 1,
    "CustomerID": 101,
    "ProductID": 456,
    "Quantity": 3,
    "Price": 9.99
}
```

