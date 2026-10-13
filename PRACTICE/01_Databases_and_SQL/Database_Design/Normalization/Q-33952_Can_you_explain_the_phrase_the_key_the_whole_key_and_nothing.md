---
type: "practice"
difficulty: "Basic"
---

# Can you explain the phrase 'the key the whole key and nothing but the key'?

**Difficulty:** Basic · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Surrogate Keys in Normalization](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

The phrase "the key, the whole key, and nothing but the key" is a familiar saying in database normalization, often used to emphasize the importance of using full candidate keys when defining primary keys. This concept ensures that every piece of information uniquely identifies a record without extraneous data. Let's break down this idea step by step.

**Comparison:**

| OrderID | CustomerID | OrderDate   | ProductID | Quantity |
|---------|------------|-------------|-----------|----------|
| 101     | 201        | 2023-05-01  | 401       | 2        |
| 102     | 201        | 2023-06-01  | 402       | 1        |

**Code:**

```text
# Example table schema
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,  # The key, the whole key, and nothing but the key
    CustomerID INT,
    OrderDate DATE,
    ProductID INT,
    Quantity INT
);
```

**Example input / output:**

```

Input:

SQL
Copy
SELECT * FROM Orders WHERE OrderID = 101;

Output:

JSON
Copy
{
    "Orders": [
        { "OrderID": 101, "CustomerID": 201, "OrderDate": "2023-05-01", "ProductID": 401, "Quantity": 2 }
    ]
}
```

**Data:**

```json
{
    "Orders": [
        { "OrderID": 101, "CustomerID": 201, "OrderDate": "2023-05-01", "ProductID": 401, "Quantity": 2 },
        { "OrderID": 102, "CustomerID": 201, "OrderDate": "2023-06-01", "ProductID": 402, "Quantity": 1 }
    ]
}
```

**Flow:**

1. Yes
2. No
3. Start
4. Define the primary key
5. Is the primary key a fullcandidate key?
6. Use it as is
7. Add more fields to make itcomplete
8. End

