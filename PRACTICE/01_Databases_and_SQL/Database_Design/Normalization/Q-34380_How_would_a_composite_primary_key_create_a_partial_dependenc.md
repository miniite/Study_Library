---
type: "practice"
difficulty: "Intermediate"
---

# How would a composite primary key create a partial dependency risk?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Surrogate Keys in Normalization](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

In database design, a composite primary key involves using multiple columns to uniquely identify a record in a table. While this approach is effective for ensuring uniqueness and integrity, it can introduce risks related to data dependencies, particularly partial dependencies. A partial dependency occurs when a non-prime attribute (i.e., an attribute that does not form part of the candidate key) depends on only a part of the primary key.

In this context, using a composite primary key can create a risk because these multiple columns might not all be equally significant in identifying a record. If a subset of the primary key is sufficient to uniquely identify some records, there’s a chance that non-prime attributes could depend solely on this subset. This scenario leads to partial dependencies and violates the 2nd Normal Form (2NF), which requires every non-key column to fully depend on the entire primary key.

For example, consider an e-commerce database with a table OrderItem, where each order can have multiple items, but not all columns form a composite primary key that uniquely identifies each item in isolation. Suppose we use (OrderID, ItemName) as our composite primary key. While this ensures uniqueness for the combination of OrderID and ItemName, it might lead to partial dependencies if ItemPrice only depends on OrderID. This violates 2NF because ItemPrice should fully depend on both OrderID and ItemName.

Let's dive into a detailed step-by-step analysis:

Understanding Composite Primary Keys: A composite primary key is used when no single column can uniquely identify all records in the table, so two or more columns are combined to form the primary key.

Identifying Partial Dependencies: To identify partial dependencies, we need to determine if any non-prime attribute depends on a part of the primary key rather than the entire key.

#### Example Scenario

Table: OrderItem
SQL
OrderID (int), ItemName (varchar), ItemPrice (decimal)
Composite Primary Key: (OrderID, ItemName)
Analysis:
OrderID and ItemName together uniquely identify an item.
However, ItemPrice only depends on OrderID. This means it violates 2NF because the dependency is not fully on the entire primary key.

Addressing the Risk:

To eliminate partial dependencies, we need to ensure that every non-prime attribute depends on the entire composite key.
One solution might be to add a unique identifier for each item within an order, ensuring ItemID forms part of the full key (OrderID, ItemName) -> (OrderID, ItemID).
SQL
OrderID (int), ItemID (int), ItemName (varchar), ItemPrice (decimal)
This change ensures that ItemPrice depends on both OrderID and ItemID, fully adhering to 2NF.

#### Closing Summary
Using a composite primary key can indeed introduce risks of partial dependencies, which violate the principles of normalization. To avoid these issues, it's crucial to ensure every non-prime attribute depends fully on the entire composite key rather than just part of it.

#### Common Mistakes
and Follow-Ups:

Mistake: Using a subset of the primary key as a dependency for a non-prime attribute.

#### Follow-Up
Discussing how to properly design the schema, perhaps by adding surrogate keys or re-evaluating the need for composite keys.

**Comparison:**

  | OrderID | ItemName | ItemPrice |
  | --- | --- | --- |
  | 1       | Laptop   | 999.99    |
  | 1       | Mouse    | 45.00     |
  | 2       | Keyboard | 150.00    |
  | 2       | Monitor  | 399.99    |

**Code:**

```sql
-- Example schema with potential partial dependencies
CREATE TABLE OrderItem (
  OrderID int,
  ItemName varchar(50),
  ItemPrice decimal,
  PRIMARY KEY (OrderID, ItemName)
);
```

**Example input / output:**

```text
graph LR;
A[OrderID=1, ItemName=Laptop] --> B(ItemPrice=999.99)
C[OrderID=1, ItemName=Mouse] --> D(ItemPrice=45.00)
E[OrderID=2, ItemName=Keyboard] --> F(ItemPrice=150.00)
G[OrderID=2, ItemName=Monitor] --> H(ItemPrice=399.99)
```

**Data:**

```json
{
  "orders": [
    {
      "order_id": 1,
      "items": [
        {"name": "Laptop", "price": 999.99},
        {"name": "Mouse", "price": 45.00}
      ]
    },
    {
      "order_id": 2,
      "items": [
        {"name": "Keyboard", "price": 150.00},
        {"name": "Monitor", "price": 399.99}
      ]
    }
  ]
}
```

**Flow:**

1. Composite Primary Key
2. Partial Dependency Risk
3. Non-Prime Attribute
4. Depends on Part of CompositeKey
5. Violates 2NF
6. Add Surrogate Key orRe-evaluate Schema

