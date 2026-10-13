---
type: "practice"
difficulty: "Basic"
---

# What is a primary key and why does every normalized table usually need one?

**Difficulty:** Basic · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Surrogate Keys in Normalization](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Surrogate_Keys_in_Normalization.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To start, let's define what a primary key (PK) is in the context of databases. A primary key is a unique identifier that uniquely identifies each record or row within a database table. It ensures data integrity and enables efficient retrieval of information.

Every normalized table typically requires a primary key because it serves multiple critical roles:

Uniqueness: Ensures that no two rows are identical.
Referential Integrity: Helps establish relationships between tables through foreign keys.
Efficient Indexing: Accelerates data lookup and query performance.

Now, let's delve deeper into the reasons why a primary key is essential for normalized tables:

Why Does Every Normalized Table Usually Need a Primary Key?
Uniqueness

A primary key ensures that each row in a table is distinct. This uniqueness constraint helps maintain data integrity by preventing duplicate entries. For example, if you have a users table with columns like user_id, name, and email, the user_id can serve as the primary key to ensure no two users have the same identifier.

Referential Integrity

In relational databases, tables are often linked through foreign keys. These foreign keys reference the primary key of another table. By having a well-defined primary key in each table, you facilitate these relationships and enforce referential integrity rules. For instance, if orders references users, it uses the user_id from the users table as its foreign key.

Efficient Indexing

Primary keys are often used to create indexes, which significantly speed up data retrieval processes. An index on a primary key makes lookups much faster compared to scanning the entire table row by row. This optimization is crucial for large datasets where performance matters.

#### Example Scenario

#### Cons
ider an e-commerce database with two tables: customers and orders.

Customers Table:

customer_id: Primary Key
name
email
address

Orders Table:

order_id: Primary Key
customer_id: Foreign Key (Referencing the customer_id in customers table)
product_id
quantity
total_price

In this setup, customer_id ensures that each customer is uniquely identified. The orders table references customer_id to establish a relationship between orders and customers.

#### Common Mistakes
Not Defining a Primary Key: Failing to define a primary key can lead to data anomalies such as duplicate entries.
Using Inappropriate Columns for PK: Choosing columns that aren’t unique, like timestamps or auto-incrementing IDs, might violate the uniqueness requirement of a primary key.

#### Closing Summary
and Follow-Ups

In summary, every normalized table needs a primary key because it ensures data integrity, enables referential relationships, and optimizes performance. While not always explicitly named as primary_key, this concept is fundamental to database design. Common pitfalls include overlooking the importance of PKs or improperly choosing columns for them.

When discussing normalization, you might be asked about higher levels like second normal form (2NF), third normal form (3NF), and so on, where primary keys play a central role in ensuring that all non-key attributes are fully dependent on the primary key.

**Code:**

```sql
-- Example of creating tables with primary keys

CREATE TABLE customers (
    customer_id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    address VARCHAR(255)
);

CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    product_id INT,
    quantity INT,
    total_price DECIMAL(10, 2),
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

**Example input / output:**

```text
Input:
- customers table with customer_id as primary key
- orders table referencing customer_id

Output:
- Orders are efficiently linked to specific customers via the foreign key in the `customer_id` column.
- This setup ensures that each order is associated with a unique customer, maintaining referential integrity.
```

**Data:**

```json
{
  "customers": [
    { "customer_id": 1, "name": "John Doe", "email": "john@example.com", "address": "123 Street, City" },
    { "customer_id": 2, "name": "Jane Smith", "email": "jane@example.com", "address": "456 Avenue, Town" }
  ],
  "orders": [
    { "order_id": 101, "customer_id": 1, "product_id": 1001, "quantity": 2, "total_price": 9.99 },
    { "order_id": 102, "customer_id": 2, "product_id": 1002, "quantity": 3, "total_price": 14.99 }
  ]
}
```

