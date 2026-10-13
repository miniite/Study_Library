---
type: "practice"
difficulty: "Deep"
---

# Why can a heavily denormalized table increase write amplification in your loading job?

**Difficulty:** Deep · **Levels:** Fresher, SDE-1/2, SDE-2/3 · [↑ Denormalization and Locking](<../../../../01_Databases_and_SQL/Database_Design/Normalization/Denormalization_Trade-offs_and_Patterns/Denormalization_and_Locking.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

TL;DR

A heavily denormalized table can lead to increased write amplification because it forces multiple writes into the database when only one piece of data changes, due to redundant copies of that data being spread across many records.

#### Triaging the Question

The question is probing your understanding of how normalization and denormalization impact write operations in a database. Specifically, you need to explain why heavily denormalized tables can result in higher write amplification during loading jobs. Write amplification refers to the situation where more writes are performed on the storage than necessary when updating or inserting data.

#### First-Principles Solution
What is Normalization and Denormalization?

Normalization involves organizing data in a database by breaking it down into multiple tables, while denormalization is the process of combining related data from different tables to reduce the number of joins required. Both techniques aim to balance storage efficiency with query performance.

Understanding Write Amplification

Write amplification occurs when an operation that should only require one write results in multiple writes due to redundant copies of data spread across the database. This can significantly increase storage usage and I/O operations, leading to decreased performance.

Why Denormalization Increases Write Amplification?

Redundant Data: In a denormalized table, data is duplicated. For example, if you have an order detail table that contains customer information redundantly, any update to the customer's address or contact details will require updates in multiple tables.

Joins Avoidance: While denormalization reduces the need for complex joins during read operations, it increases the complexity of write operations because every piece of data must be updated across multiple records.

Data Consistency Issues: Ensuring data consistency across all copies can be challenging and often requires additional logic or triggers, further complicating writes.

Step-by-Step Analysis
Identify Redundant Data: Consider a denormalized table where customer information is duplicated in several tables.
Single Change Requires Multiple Writes: If the address of a specific customer changes, you need to update all instances of that customer's address across multiple records and possibly related tables.

#### Example Scenario
Suppose we have two tables: Orders (denormalized) and Customers.
An order in Orders references a customer ID, and the customer details are also stored redundantly in the Orders table.
Changing the address of the customer requires updating the Customers table AND all relevant records in the Orders table.
Detailed Example

#### Cons
ider an Orders table with redundant customer information:

SQL
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    product_name VARCHAR(255),
    quantity INT,
    customer_id INT,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    address VARCHAR(255)
);

To update the address of a specific customer, you would need to:

Locate the relevant order(s) in the Orders table.
Update each order with the new address.
Why Normalization Can Be Better?

Normalization reduces write amplification by ensuring that data is stored only once and updating a single record when necessary. This minimizes the number of writes required, especially for large datasets.

#### Closing Summary

In summary, heavily denormalized tables can increase write amplification because they spread redundant copies of data across multiple records, leading to multiple write operations even for small updates. Normalization, on the other hand, consolidates related data into a single record, reducing the number of writes and improving efficiency.

#### Common Mistakes/Follow-Ups
Ignoring Data Consistency: Not implementing proper mechanisms to ensure that changes are propagated correctly across all copies can lead to inconsistencies.
Over-Engineering: Denormalization should be done judiciously; excessive denormalization can make the database structure overly complex and harder to maintain.

For a more detailed analysis, you might want to consider the specific normalization levels (1NF, 2NF, 3NF) and how they impact write operations.

**Comparison:**

```
Yes
No
Orders
Customer ID Matches?
Update Order Address
Fetch Orders with CustomerID= 12345
Update Each Order's Address
Address Updated
```

**Code:**

```sql
CREATE TABLE Orders (
    order_id INT PRIMARY KEY,
    product_name VARCHAR(255),
    quantity INT,
    customer_id INT,
    first_name VARCHAR(255),
    last_name VARCHAR(255),
    address VARCHAR(255)
);
```

**Example input / output:**

```
Input: Update the address of customer with ID 12345 to "456 Elm St".
Output:
JSON
Copy
{
  "order_id": 101,
  "product_name": "Laptop",
  "quantity": 1,
  "customer_id": 12345,
  "first_name": "John",
  "last_name": "Doe",
  "address": "456 Elm St"
}
```

**Data:**

```json
{
  "order_id": 101,
  "product_name": "Laptop",
  "quantity": 1,
  "customer_id": 12345,
  "first_name": "John",
  "last_name": "Doe",
  "address": "123 Main St"
}
```

**Flow:**

1. Yes
2. No
3. Input: Update CustomerAddress
4. Choose Denormalized Table
5. Customer ID Matches?
6. Update Single Record in Table
7. Query for Relevant Records
8. Update Each Record
9. Address Updated

