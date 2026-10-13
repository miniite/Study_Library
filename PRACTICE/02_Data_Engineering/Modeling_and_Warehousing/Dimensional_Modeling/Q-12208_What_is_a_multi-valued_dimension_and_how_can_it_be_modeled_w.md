---
type: "practice"
difficulty: "Deep"
---

# What is a multi-valued dimension and how can it be modeled with a bridge table?

**Difficulty:** Deep · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Fact and Dimension Keys](<../../../../02_Data_Engineering/Modeling_and_Warehousing/Dimensional_Modeling/Fact_and_Dimension_Keys.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To model a multi-valued dimension in data modeling, we need to understand that traditional dimensions like Customer or Product often have single values (e.g., one customer per record). However, some dimensions can logically contain multiple values. For example, an order might involve multiple payment methods or shipping addresses. This is where the concept of a bridge table comes into play. A multi-valued dimension is modeled using a central fact table and bridge tables that connect these many-to-many relationships.

Let's start by understanding what we're tackling: you have a traditional star schema with facts and dimensions, but some of your dimensions need to support multiple values. The key idea here is that we can't directly store multiple values in the dimension itself (as databases don't allow arrays or lists as primary keys), so we use a bridge table.

To illustrate, consider an e-commerce application where each order can have multiple payment methods and shipping addresses. This would require us to model these relationships with additional tables.

#### Triaging the Question

The question is essentially asking how to handle many-to-many relationships in a dimensional model. The interviewer might be probing your understanding of:

Data normalization: How you ensure data integrity.
ETL processes: How you populate and maintain this structure.
Query performance considerations: How these models affect query performance.

#### First-Principles Solution
Step 1: Identify the Dimensions

First, identify which dimension is multi-valued. In our example, we have:

Orders: The fact table capturing orders.
Payment Methods: A dimension representing different payment methods (e.g., Credit Card, PayPal).
Shipping Addresses: Another dimension for managing shipping addresses.
Step 2: Create a Central Fact Table

The central fact table contains the primary keys of both the Orders and the multi-valued dimensions. For instance:

SQL
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    -- other order details
);
Step 3: Create Bridge Tables

For each multi-valued dimension, create a bridge table that connects the Orders table to the respective dimension.

Payment Methods Bridge Table:
SQL
CREATE TABLE Order_PaymentMethods (
    OrderID INT,
    PaymentMethodID INT,
    PRIMARY KEY (OrderID, PaymentMethodID),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (PaymentMethodID) REFERENCES PaymentMethods(PaymentMethodID)
);
Shipping Addresses Bridge Table:
SQL
CREATE TABLE Order_ShippingAddresses (
    OrderID INT,
    ShippingAddressID INT,
    PRIMARY KEY (OrderID, ShippingAddressID),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ShippingAddressID) REFERENCES ShippingAddresses(ShippingAddressID)
);
Step 4: Querying the Model

When querying this model, you would typically use JOINs to retrieve all relevant details. For example:

SQL
SELECT o.*, pm.PaymentMethod, sa.Address
FROM Orders o
LEFT JOIN Order_PaymentMethods opm ON o.OrderID = opm.OrderID
LEFT JOIN PaymentMethods pm ON opm.PaymentMethodID = pm.PaymentMethodID
LEFT JOIN Order_ShippingAddresses osa ON o.OrderID = osa.OrderID
LEFT JOIN ShippingAddresses sa ON osa.ShippingAddressID = sa.ShippingAddressID;

#### Closing Summary
and Common Mistakes/Follow-Ups

By using bridge tables, you can effectively manage multi-valued dimensions in a star schema. However, this approach adds complexity to the ETL process and can impact query performance if not optimized properly.

#### Common Mistakes
Not Using Bridge Tables: Failing to recognize that many-to-many relationships need special handling.
Inadequate Indexing: Not adding appropriate indexes on bridge tables can degrade performance.
Ignoring Cardinality Issues: Poorly designed models might lead to overly complex queries or excessive data redundancy.
Self-Check

The solution is correct and covers the key points of modeling multi-valued dimensions with bridge tables, including steps for identification, table creation, querying, and potential pitfalls.

**Comparison:**

```
ERDiagram
    Orders ||--o{ Order_PaymentMethods : order_id (FK)
    Orders ||--o{ Order_ShippingAddresses : order_id (FK)
    PaymentMethods ||--| Order_PaymentMethods : payment_method_id (PK, FK)
    ShippingAddresses ||--| Order_ShippingAddresses : shipping_address_id (PK, FK)
```

**Code:**

```sql
CREATE TABLE Orders (
    OrderID INT PRIMARY KEY,
    -- other order details
);

CREATE TABLE PaymentMethods (
    PaymentMethodID INT PRIMARY KEY,
    MethodName VARCHAR(50)
);

CREATE TABLE ShippingAddresses (
    ShippingAddressID INT PRIMARY KEY,
    AddressLine1 VARCHAR(255),
    AddressLine2 VARCHAR(255),
    City VARCHAR(100)
);

CREATE TABLE Order_PaymentMethods (
    OrderID INT,
    PaymentMethodID INT,
    PRIMARY KEY (OrderID, PaymentMethodID),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (PaymentMethodID) REFERENCES PaymentMethods(PaymentMethodID)
);

CREATE TABLE Order_ShippingAddresses (
    OrderID INT,
    ShippingAddressID INT,
    PRIMARY KEY (OrderID, ShippingAddressID),
    FOREIGN KEY (OrderID) REFERENCES Orders(OrderID),
    FOREIGN KEY (ShippingAddressID) REFERENCES ShippingAddresses(ShippingAddressID)
);
```

**Example input / output:**

```text
-- Example query to retrieve orders with payment methods and shipping addresses
SELECT o.*, pm.MethodName AS PaymentMethod, sa.AddressLine1, sa.City
FROM Orders o
LEFT JOIN Order_PaymentMethods opm ON o.OrderID = opm.OrderID
LEFT JOIN PaymentMethods pm ON opm.PaymentMethodID = pm.PaymentMethodID
LEFT JOIN Order_ShippingAddresses osa ON o.OrderID = osa.OrderID
LEFT JOIN ShippingAddresses sa ON osa.ShippingAddressID = sa.ShippingAddressID;
```

**Data:**

```json
{
  "Orders": [
    { "OrderID": 1 },
    { "OrderID": 2 }
  ],
  "PaymentMethods": [
    { "PaymentMethodID": 1, "MethodName": "Credit Card" },
    { "PaymentMethodID": 2, "MethodName": "PayPal" }
  ],
  "ShippingAddresses": [
    { "ShippingAddressID": 101, "AddressLine1": "123 Elm St", "City": "Springfield" },
    { "ShippingAddressID": 102, "AddressLine1": "456 Oak Ave", "City": "Springfield" }
  ],
  "Order_PaymentMethods": [
    { "OrderID": 1, "PaymentMethodID": 1 },
    { "OrderID": 1, "PaymentMethodID": 2 },
    { "OrderID": 2, "PaymentMethodID": 1 }
  ],
  "Order_ShippingAddresses": [
    { "OrderID": 1, "ShippingAddressID": 101 },
    { "OrderID": 1, "ShippingAddressID": 102 },
    { "OrderID": 2, "ShippingAddressID": 101 }
  ]
}
```

**Flow:**

1. Order
2. Order_PaymentMethods
3. Order_ShippingAddresses
4. PaymentMethods
5. ShippingAddresses

