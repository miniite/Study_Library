---
type: "practice"
difficulty: "Basic"
---

# What are the requirements for a table to be in Second Normal Form?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ First Normal Form](<../../../../01_Databases_and_SQL/Database_Design/Normalization/First_Normal_Form.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To ensure a table is in Second Normal Form (2NF), it's essential to understand that 2NF addresses certain data integrity issues by building upon the principles of First Normal Form (1NF). A table is said to be in 2NF if and only if:

It meets all the criteria for 1NF, which means every column contains atomic values.
All non-key columns are fully functionally dependent on the primary key.

In simpler terms, a table should not have any partial dependencies, meaning that each piece of data (column) must be completely determined by the entire primary key. This prevents the leakage of redundant data into the table structure, ensuring that each row is independent and complete with respect to the primary key.

**Code:**

```sql
-- Example of a table in 2NF
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    CustomerID INT,
    OrderDate DATE,
    OrderTotal DECIMAL(10, 2),
    ShippingAddress VARCHAR(255)
);
```

