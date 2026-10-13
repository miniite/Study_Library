---
type: "practice"
difficulty: "Intermediate"
---

# How does normalization affect the number of JOINs in a query?

**Difficulty:** Intermediate · **Levels:** Architect/Principal, Fresher, SDE-1, SDE-1/2, SDE-2/3 · [↑ Normalization Fundamentals](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Normalization_Fundamentals.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

Normalization is a process used to organize data into tables by eliminating redundant information and ensuring that each column contains atomic (indivisible) values. This helps prevent anomalies such as insertion, update, deletion, and data redundancy issues. Denormalization, on the other hand, involves adding redundant data to improve query performance.

In this context, we need to understand how normalization affects the number of JOINs in a query. As you move from lower to higher normal forms (1NF to 3NF), each step introduces more constraints and rules, which can increase or decrease the complexity of queries, particularly the use of JOIN operations.

To illustrate, consider a simple example: a database with three tables—Customers, Orders, and OrderDetails. Initially, these are denormalized, meaning some data is repeated across multiple rows. When normalized, we might split this into five tables:

Customers (CustomerID, Name)
Orders (OrderID, CustomerID, OrderDate)
OrderDetails (OrderDetailID, OrderID, ProductID, Quantity)
Products (ProductID, ProductName, Price)
Addresses (AddressID, CustomerID, AddressLine1, AddressLine2)

Let's see how the normalization affects JOINs in a query.

#### Triaging the Question

The core of this question is to understand the relationship between normalization and the number of JOIN operations required to retrieve data from multiple tables. The interviewer might be probing your understanding of:

The benefits and trade-offs of different normal forms.
How normalized data can increase the complexity of queries due to the need for more JOINs.
Step-by-Step Solution
1. Understanding Normalization Levels
1NF: Eliminate duplicate columns from each table. (Ensures atomic values, but doesn't eliminate redundancy)
2NF: Adds a requirement that all non-key fields are fully dependent on the primary key.
3NF: Ensures that there is no transitive dependency—non-key fields should only be dependent directly on the primary key.
2. Impact of Normalization on JOINs
1NF to 3NF: As you move towards more normalized forms, relationships between entities are broken down into smaller tables.
Each table in a highly normalized schema has fewer columns and potentially less data redundancy but requires more joins to retrieve related information.
For example, retrieving all orders along with the customer name would require multiple JOINs.
3. Example

Let's consider a query that retrieves all customers who placed an order for a specific product:

SQL
SELECT C.Name, O.OrderDate, OD.Quantity, P.ProductName
FROM Customers C
JOIN Orders O ON C.CustomerID = O.CustomerID
JOIN OrderDetails OD ON O.OrderID = OD.OrderID
JOIN Products P ON OD.ProductID = P.ProductID
WHERE P.ProductName = 'Widget';

In this query:

We need to JOIN Customers, Orders, and OrderDetails tables.
The product information is fetched from the Products table using a separate JOIN.
4. Denormalization for Performance

To reduce the number of JOINs, denormalization can be used. For example, you could add a column ProductName to the Orders table and store the relevant product details there:

SQL
ALTER TABLE Orders ADD ProductName VARCHAR(100);

Now, the query can be simplified to:

SQL
SELECT C.Name, O.OrderDate, O.Quantity, O.ProductName
FROM Customers C
JOIN Orders O ON C.CustomerID = O.CustomerID
WHERE O.ProductName = 'Widget';

This reduces the complexity of the JOIN operation and potentially improves performance.

#### Closing Summary
+ Common Mistakes/Follow-Ups

Normalization is crucial for data integrity but can increase query complexity, particularly when dealing with multiple related tables. Denormalization can be a trade-off to improve query performance at the cost of increased storage space and potential data redundancy issues.

#### Common mistakes

Over-normalizing without considering the impact on query performance.
Failing to denormalize appropriately for frequent read-heavy operations.

#### Follow-ups
might include discussing specific normalization rules, trade-offs in real-world scenarios, or optimization strategies for both normalized and denormalized schemas.

**Comparison:**

    |CustomerID| Name|
    |OrderID| CustomerID| OrderDate| ProductID| Quantity|
    |ProductID| ProductName|

**Code:**

```sql
-- Example of a 1NF table
CREATE TABLE Orders (
    OrderID INT,
    CustomerID INT,
    OrderDate DATE,
    ProductID INT,
    Quantity INT
);

-- Example of a 2NF table (Assuming all non-key fields are dependent on the primary key)
ALTER TABLE Orders ADD CONSTRAINT FK_Customer FOREIGN KEY (CustomerID) REFERENCES Customers(CustomerID);
```

**Example input / output:**

```text
-- Example query for normalized schema
SELECT C.Name, O.OrderDate, OD.Quantity, P.ProductName
FROM Customers C
JOIN Orders O ON C.CustomerID = O.CustomerID
JOIN OrderDetails OD ON O.OrderID = OD.OrderID
JOIN Products P ON OD.ProductID = P.ProductID
WHERE P.ProductName = 'Widget';

-- Example query for denormalized schema
SELECT C.Name, O.OrderDate, O.Quantity, O.ProductName
FROM Customers C
JOIN Orders O ON C.CustomerID = O.CustomerID
WHERE O.ProductName = 'Widget';
```

**Data:**

```json
{
  "Customers": [
    { "CustomerID": 1, "Name": "John Doe" },
    { "CustomerID": 2, "Name": "Jane Smith" }
  ],
  "Orders": [
    { "OrderID": 101, "CustomerID": 1, "OrderDate": "2023-10-01", "ProductID": 101, "Quantity": 5 },
    { "OrderID": 102, "CustomerID": 2, "OrderDate": "2023-10-02", "ProductID": 102, "Quantity": 3 }
  ],
  "Products": [
    { "ProductID": 101, "ProductName": "Widget" },
    { "ProductID": 102, "ProductName": "Gadget" }
  ]
}
```

**Flow:**

1. Denormalized Schema
2. Customers
3. Orders
4. OrderDetails
5. Products

