---
type: "practice"
difficulty: "Intermediate"
---

# How would you use a surrogate key in a dimension table to protect against changing natural keys from the source?

**Difficulty:** Intermediate · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Surrogate Keys in Normalization](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In data warehousing and ETL processes, using a surrogate key in a dimension table is crucial for maintaining data integrity, especially when dealing with changing natural keys from the source. A surrogate key serves as a unique identifier for each record in the dimension table, independent of any external factors like customer names or addresses that might change over time.

The primary goal here is to ensure that historical data remains consistent and unaltered even if the source system updates its natural keys. By using a surrogate key, we can track changes without affecting the integrity of existing records.

**Code:**

```sql
-- Example SQL Table Creation for Dimension with Surrogate Key

CREATE TABLE Customers (
    CustomerID INT PRIMARY KEY AUTO_INCREMENT,
    CustomerNumber VARCHAR(20) UNIQUE NOT NULL, -- Natural Key
    FirstName VARCHAR(50),
    LastName VARCHAR(50),
    Email VARCHAR(100)
);
```

**Example input / output:**

```text
Input:
Original Customer Record:
- CustomerID: 1
- CustomerNumber: CUST-001
- FirstName: John
- LastName: Doe

Updated Customer Record from Source:
- New CustomerNumber: NEWCUST-001 (due to a change in the source system)

Output:
- Original Record remains unchanged, preserving historical data.
- New Record added with updated key.
```

**Data:**

```json
{
  "Customer": [
    {
      "CustomerID": 1,
      "CustomerNumber": "CUST-001",
      "FirstName": "John",
      "LastName": "Doe",
      "Email": "john.doe@example.com"
    },
    {
      "CustomerID": 2,
      "CustomerNumber": "CUST-002",
      "FirstName": "Jane",
      "LastName": "Smith",
      "Email": "jane.smith@example.com"
    }
  ]
}
```

